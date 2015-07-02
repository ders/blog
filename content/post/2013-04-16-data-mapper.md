+++
date = "2013-04-16T12:19:00+09:00"
title = "Data Mapper"

+++

I've been working on a project with [Padrino](http://www.padrinorb.com/) and [Data Mapper](http://datamapper.org).
So far I'm quite a fan of the Data Mapper way of doing things.

Unfortunately, I'm schooled in the old way -- ugly messes of `SELECT`, `JOIN` and `ORDER BY`,
intelligible only by SQL gurus and dependent not just on SQL but on a particular variety of such, which in my case would be MySQL.

I label this as unfortunate because my concept of database (e.g. MySQL) heavily influences how I organize the code I write.
I often find myself with a "How do you do this in DataMapper?" kind of question, where *this* is something that I know how to do the old way.
After all, DataMapper is generating SQL queries from the models I write, so if it's easy in SQL, shouldn't it also be easy in DataMapper?

(Side note: DataMapper doesn't *necessarily* generate SQL, but in my current project the backend is SQL and I see the generated queries on the debugging console.)

Recently I've a question of this sort that I haven't been able to solve.

I have a model with an ordering field, which we'll call `position`.  I want to sort by this field (ascending), except that I want all the zeroes to be at the end.  In addition, I'd like the all the records with `position=0` sorted by `id` descending.

In MySQL, I would write:

{{< highlight sql >}}
SELECT * FROM `things` ORDER BY `position`=0, `position`, `id` DESC
{{< /highlight >}}

Free donuts to the first person who can make a DataMapper version of this.
