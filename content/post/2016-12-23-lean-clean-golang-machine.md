+++
Description = ""
Tags = []
date = "2016-12-23T14:49:00+09:00"
title = "A lean, clean Golang machine"

+++
Writing a [Go](https://golang.org/) package that interacts with a relational
data store such as Postgres is full of messiness.

Those of us who appreciate the strong-typedness of Go probably also appreciate
the strong-typedness of SQL, and vice versa.  Unfortunately, communication
between Go and SQL is less than ideal. This is due partly to the mostly
free-form text format of data exchange (queries) and partly to some subtle
differences in data types.

Database nulls are a particular headache, leading to the contortions of defining
types such [NullString](https://golang.org/pkg/database/sql/#NullString),
[NullInt64](https://golang.org/pkg/database/sql/#NullInt64), and
[NullBool](https://golang.org/pkg/database/sql/#NullBool), and an extra check
is required every time you want distinguish a null from a zero value.

Why not use an ORM? There has been
[a lot written](http://www.hydrogen18.com/blog/golang-orms-and-why-im-still-not-using-one.html)
[on this already](https://blog.codinghorror.com/object-relational-mapping-is-the-vietnam-of-computer-science/),
but in a nutshell, the level of generality required means that
[pretty much everything is an interface{}](https://godoc.org/github.com/jinzhu/gorm)
with runtime checks to cast stuff into the types you need, and at this point
we've lost the benefits of Go's strong typing and may as well write our whole
application in Ruby.

I've found that programmers who appreciate the power and control that comes from
writing in a low-level compiled language such as Go also appreciate the power
can control that comes from writing queries yourself in SQL.

## So what's the problem, really?

The real headache of [Go + SQL](https://golang.org/pkg/database/sql/) is the
volume of boilerplate code that goes with even relatively simple operations.

(1) Run a query that doesn't return any results.

```
_, err := db.Exec(query, ...args)
if err != nil {
	return err
}
```

(1a) Run a query that doesn't return any results, but we want to know how many
rows were changed.

```
res, err := db.Exec(query, ...args)
if err != nil {
	return err
}
count, err := res.RowsAffected()
if err != nil {
	return err
}
```

(1b) Run a query that doesn't return any results, and we'd like to catch and
process integrity violations (e.g. duplicate entry on a unique field). This one
requires some database-specific code; the example here is for Postgres.

```
_, err := db.Exec(query, ...args)
duplicate := false
if err != nil {
	if pgerr, ok := err.(*pq.Error); ok {
		duplicate = pgerr.Code.Class().Name() == "integrity_constraint_violation"
	}
	if !duplicate {
		return err
	}
}
```

(1c) Run a query that doesn't return any results, and we'd like to catch and
process data exceptions (e.g. number out of range). This uses the same strategy as 1b and can be combined with it.

(2) Run a query that returns one row.

```
err := db.QueryRow(query, ...args).Scan(&arg1, &arg2, ... )
if err != nil {
	return err
}
```

(2a) Run a query that returns one row, and we'd like to catch and process the
case where no rows are returned.

```
err := db.QueryRow(query, ...args).Scan(&arg1, &arg2, ... )
noRows := err == ErrNoRows
if err != nil && !noRows {
	return err
}
```

(3) Run a query that returns multiple rows.

```
rows, err := db.Query(query, ...args)
if err != nil {
	return err
}
defer rows.Close()
for rows.Next() {
	err := rows.Scan(&arg1, &arg2, ... )
	if err != nil {
		return err
	}
}
err = rows.Err()
if err != nil {
	return err
}
```

None of these is particularly bad as far as boilerplate goes, but unless we're
writing an ORM (and we've already decided we're not), we're going to have tens,
perhaps hundreds of these scattered throughout our application.  Add to that an
other `if err != nil` every time we start a transaction, and I'm thinking
there's got to be a better way.

## Organizing database access around high-level functionality

We would like to follow the
[unit of work](http://martinfowler.com/eaaCatalog/unitOfWork.html)
pattern and create something akin to the
[session model](http://docs.sqlalchemy.org/en/latest/orm/session_basics.html)
of SQLAlchemy.

A simple example of a unit of work is a password reset, which checks for an
email match, and then generates, saves, and returns a reset code. This will
involve a minimum of two queries, which need to be in the same transaction.
(Much more complicated units of work are possible, of course, both read-only
and read-write.)

Our goal then is to find a way to have just one copy of all the boilerplate above and be able to substitute queries and argument lists as needed.

I'm going to propose that it's straightforward to implement such a thing Go
by defining a custom transaction handler which extends
[the one in database/sql](https://golang.org/pkg/database/sql/#Tx).
This is done within the package that uses it.

```
type Tx struct {
	sql.Tx
}
```

We extend `sql.Tx` with methods to (a) convert all database errors to panics so
that we can catch and process them all in one place, and (b) easily iterate over
result sets.

To accomplish (a), we add the methods `MustExec`, `MustQuery`, and `MustQueryRow`.
These are identical to `Exec`, `Query`, and `QueryRow` except that they panic
instead of returning an error code. Also, in the case of `MustQuery` and `MustQueryRow`,
they return custom `Rows` and `Row` objects that have similar extensions.

To accomplish (b), we add the method `Each` to the custom `Rows` object returned
by `MustQuery`.  Method `Each` iterates over the result set and calls a
callback function for each row.

The `ourError` type is used to wrap errors that we want to convert back to error
codes. It distinguishes them from other kinds of panics (e.g. out of memory).

```
type ourError struct {
	err error
}

func (tx Tx) MustExec(query string, args ...interface{}) sql.Result {
	res, err := tx.Exec(query, args...)
	if err != nil {
		panic(ourError{err})
	}
	return res
}

func (tx Tx) MustQuery(query string, args ...interface{}) *Rows {
	rows, err := tx.Query(query, args...)
	if err != nil {
		panic(ourError{err})
	}
	return &Rows{*rows}
}

func (tx Tx) MustQueryRow(query string, args ...interface{}) *Row {
	row := tx.QueryRow(query, args...)
	return &Row{*row}
}
```

The custom `Row` and `Rows` types are defined analogously.
`Row` is extended with a `MustScan` method:

```
type Row struct {
	sql.Row
}

func (row Row) MustScan(args ...interface{}) {
	err := row.Scan(args...)
	if err != nil {
		panic(ourError{err})
	}
}
```

`Rows` is extended with a `MustScan` method and also with the `Each` iterator
described above.

```
type Rows struct {
	sql.Rows
}

func (rows Rows) MustScan(args ...interface{}) {
	err := rows.Scan(args...)
	if err != nil {
		panic(ourError{err})
	}
}

func (rows *Rows) Each(f func(*Rows)) {
	defer rows.Close()
	for rows.Next() {
		f(rows)
	}
	err := rows.Err()
	if err != nil {
		panic(ourError{err})
	}
}
```

Now to make it all work, we define a custom transaction function.  It
sets up the transaction, provides the custom transaction handler to our
callback, and then catches the panics.

```
func Xaction(db *sql.DB, f func(*Tx)) (err error) {

	var tx *sql.Tx
	tx, err = db.Begin()
	if err != nil {
		return
	}

	defer func() {
		if r := recover(); r != nil {
			if ourerr, ok := r.(ourError); ok {
				// This panic of from tx.Fail() or the equivalent.  Unwrap it,
				// process it, and return it as an error code.
				tx.Rollback()
				err = ourerr.err
				if err == sql.ErrNoRows {
					err = ErrDoesNotExist
				} else if pgerr, ok := err.(*pq.Error); ok {
					switch pgerr.Code.Class().Name() {
					case "data_exception":
						err = ErrInvalidValue
					case "integrity_constraint_violation":
						// This could be lots of things: foreign key violation,
						// non-null constraint violation, etc., but we're generally
						// checking those in advance. As long as our code is in
						// order, unique constraints will be the only things we're
						// actually relying on the database to check for us.
						err = ErrDuplicate
					}
				}
			} else {
				// not our panic, so propagate it
				panic(r)
			}
		}
	}()

	f(&Tx{*tx}) // this runs the queries

	tx.Commit()
	return
}
```

This covers all of our boilerplate needs except for (1a) above.
To accommodate (1a), we could extend `sql.Result` the same way we extended
the others, but I haven't really needed it yet, so I'll leave it as an
exercise for the reader,

One final method that's there just to make everything neat and tidy is a `Fail`
method on the transaction which can be used to return an arbitrary error.

```
func (tx Tx) Fail(err error) {
	panic(ourError{err})
}
```

## The result

Our application code is now a lot neater.

```
err := Xaction(func(tx *Tx) {

	// Run a query that doesn't return any results.
	tx.MustExec(query1, ...args)

	// Run a query that returns one row.
	tx.MustQueryRow(query2, ...args).MustScan(&arg1, &arg2, ... )

	// Run a query that returns multiple rows.
	tx.MustQuery(query3, ...args).Each(func(r *Rows) {
		r.MustScan(&arg1, &arg2, ... )
	})
})

if err != nil {
	switch err {

	case ErrDoesNotExist:
		// query2 returned no rows

	case ErrInvalidValue:
		// data exception

	case ErrDuplicate:
		// integrity violation

	default:
		return err
	}
}
```

And since this is an extension to the stock transaction handler rather than
a replacement for it, we can still use the original non-must methods for
any edge case that might require a different kind of error handling.
