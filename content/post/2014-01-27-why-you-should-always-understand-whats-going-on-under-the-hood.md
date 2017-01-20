+++
date = "2014-01-27T10:07:00+09:00"
title = "Why you should always understand what's going on under the hood"

+++

This is about how I got bitten by [DataMapper](http://datamapper.org/).
No infection and I didn't get rabies, but it still hurt.

I should mention that overall I think DataMapper is pretty good.
It has most of the core features that an ORM should have without unneeded complexity.
It beats the pants off of Active Somethingorother when it comes to making clean, backend-independent models.

Unfortunately _most of the core features_ isn't quite _all of the core features_, and when you really need that one missing piece, that's when the trouble starts.

What bit me was the combination of two unrelated things:

- I needed an atomic update with conditions attached.
- Datamapper has different representations of the Boolean type, even between different SQLs.

### The Missing Feature

The glaringly missing feature is the ability to update a value and know
whether or not we actually changed anything.

In SQL this would look like this:

```
UPDATE "quarks" SET "seen" = 't', "whosaw" = 'ders' WHERE "id" = 42 AND "seen" = 'f'
```

Then I would get the number of affected rows to see if a change was made
(i.e. whether or not the quark had already been seen).

So let's try it with DataMapper:

```
Quark.all(:id => 42, :seen => false).update(:seen => true, :whosaw => 'ders')
```

Two problems here.
The first problem is that the return value is always true, and we don't
get to see how many rows were updated.
This isn't too big a deal;
we can work around it by using `Quark.first` instead of `Quark.all`,
generating an exception if no records are found.

The second problem is the dealbreaker.
Datamapper insists on generating two separate queries for the single
update statement:

```
SELECT "id", "size", "name", "seen", "whosaw" FROM "quarks" WHERE ("id" = 42 AND "seen" = 'f') ORDER BY "id"
UPDATE "quarks" SET "seen" = 't', "whosaw" = 'ders' WHERE "id" = 42
```

This obviously won't do, as it's not thread-safe.
Two users running this code at the same time would both believe that they saw the quark first.

### The Solution

The solution was to write the update query in SQL.

```
da = DataMapper.repository(:default).adapter
r = da.execute("UPDATE quarks SET seen='t', whosaw='ders' WHERE id=42 AND seen='f'")
if r.affected_rows > 0
   ... # we saw it first
```

Kind of defeats the purpose of having an ORM, but it gets the job done.
And as it turns out, I'm
[not the first one](http://stackoverflow.com/questions/18650932/how-to-add-a-where-clause-in-update-query-in-datamapper)
to run into this issue.

### How I Got Bitten

Little did I know that the internal mapping of a Boolean field varies
between SQL implementations.
For Sqlite and Postgres, it's a character field with `'t'` and `'f'` values, whereas for MySQL it's the integers 1 and 0.

In my case the unit tests all passed, but the live server (with a MySQL backend)
started returning 500s.

It's easy enough to change the query to work with MySQL:

```
r = da.execute("UPDATE quarks SET seen=1, whosaw='ders' WHERE id=42 AND seen=0")
```

But then the unit tests fail.

In the end, I wrote this bit of horrible code to keep the tests passing
and the live server happy.

```
t, f = if da.options['scheme'] == 'sqlite'
  ["'t'", "'f'"] # sqlite
else
  [1, 0] # mysql
end

  ...

r = da.execute("UPDATE quarks SET seen=#{t}, whosaw='ders' WHERE id=42 AND seen=#{f}")
```

(There may be a way to extract `t` and `f` directly from the
DataMapper internals, but I'm not that good yet.)
