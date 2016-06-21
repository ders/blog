+++
Description = ""
Tags = []
date = "2016-06-21T17:20:00+09:00"
title = "CRUD APIs are crud"

+++
## CRUD APIs are crud

I'm making the case specifically about [REST](http://web.archive.org/web/20130116005443/http://tomayko.com/writings/rest-to-my-wife) APIs, but in fact everything here applies to any API, REST or not.

It's a common paradigm to create a data model as a collection of tables in a relational database and then access the data from some client app (mobile or web).  [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) has become a popular way to access the data, perhaps because it's easy to make and easy to explain.

In CRUD, we're essentially giving the caller direct access to INSERT, SELECT, UPDATE and DELETE commands on our SQL database.  Or something analogous if you're into NoSQL.  It comes with some permissions checking, of course, but as far as the capabilities of the API, that's pretty much it.

The worst thing this does is expose the schema to the client, making it difficult to change the internal structure later on.  Want to fix how tags are stored?  Too bad, you're going to break the API.

Besides that, there's a lot of [database capability](http://www.agiledata.org/essays/relationalDatabases.html#AdvancedFeatures) that's missing.

What happens when we have some business logic, e.g. in a stored procedure?  We'll have to create a separate endpoint for that.  

What happens when we have some limited resource that we need to allocate on a first-come, first-served basis, e.g. room reservations.  Again, we need some special processing to ensure that only one of two simultaneous requests succeed.

What happens when we need some concept of transactions, that when a series of operations can't be completed we revert back to the original state?  Once again, we need to handle this separately.

What happens when we need to enforce some consistency between tables?  In the case of foreign key constraints, it's usually enough just to do the updates in the proper order, but other more complicated constraints will either need their own separate endpoints or will need to be momentarily violated.  And being violated is never acceptable, even for just a moment.

The biggest problem with a CRUD API is that it's [shifting all the business logic to the caller](https://lostechies.com/chrispatterson/2014/01/03/crud-is-not-a-service/), whereas it should instead be invisible to the caller.  Even Microsoft [recognized CRUD as an anti-pattern](https://msdn.microsoft.com/en-us/library/ms954638.aspx#soade_topic3), and that was way back in 2005.  Even when we're only doing read and display, it's often necessary to make several API calls to produce one document, unnecessarily slowing down load times.

The second-biggest problem with a CRUD API is specific to the update operation.  Update does not represent any realistic use case.  When do you ever want to rewrite an entire database row?  We carry this mistake all the way to the UI, where we press `edit` on our profile, get back all of our data in input fields, change one field, and then write everything back.

## APIs that work

I'm proposing a way to approach APIs, a way that avoids the pitfalls of CRUD.  If you're practicing [domain-driven design (DDD)](http://dddcommunity.org/learning-ddd/what_is_ddd/), this will happen naturally.  (Side note: at our company, we've been using DDD since day one, but no one here knew there was a buzzword for it.)  None of what I'm proposing is new or groundbreaking; it's just the way we should be doing things.

For read operations, there is one API call per display operation.  Everything needed to render the requested view comes back as one bundle.  Dynamic web content that's generated server-side is done this way, and the API can too.  As a bonus, we can use the same API as internal for server-generated pages and as external for client-generated views.

For write operations, there is a one-to-one correspondence between a user action and an API call.  On the backend, one API call is one transaction, and if any part fails, then the whole thing fails.  (Side note: one should never, ever build a system where it's possible for only part of a user action to succeed.  Usability nightmare.)

If we absolutely need some CRUD-style functionality (e.g. updating one's profile), we should make our updates one field at a time.  Not only does this match more closely what the average user will be doing, but it gives us an easy way to manage concurrency: simply require an update call to specify both the old and new value.  If the old value doesn't match, it's an error.

## Tracking changes and archiving

Tracking changes and archiving are two capabilities that are often added to a data store as an afterthought.  I'd like to be proactive and incorporate them into the data design from the beginning.

The simplest way to track changes is with created-at and updated-at fields on every db model, and most database engines have neat ways to auto-update these fields.  This level of tracking is of limited use, however, as we don't know what changed or who changed it.

There are plenty of add-ons to do detailed revision tracking ([django-reversion](https://django-reversion.readthedocs.io/en/stable/) is one I like), but I'm a little bit concerned about the performance hit.  Also, such add-ons make the created-at and updated-at fields redundant.  That's probably a good thing.

As for archiving, a common technique is to add a boolean field called `archived` to every model you want to be able to archive.  On this plus side, it's easy not to break references when you have non-archived data that refers to archived data, but [we really shouldn't have that happening](https://en.wikipedia.org/wiki/Design_smell).  On the minus side, we end up adding `and not archived` to nearly every query.

We also might want to be able to permanently delete some archived material after a certain expiration time.  We'd then need an `archived_at` field as well.

Here's where CRUD fails again:  Archive a record by setting `archived` to true and write it back.  Unarchive it similarly.  Determine the age of data by reading the created-at and updated-at fields on the model.

I propose that archiving and revision tracking can be implemented together in a way that's clean and transparent to the client.

Instead of adding extra fields to the models, all the archive and tracking information goes into a read/append-only journal, which may or may not be implemented as a database table.

The journal contains one entry for each user action (see above). If there are system actions (e.g. daily aggregations) that get written to the database, those get included as well.  Each entry contains a before-and-after detail of all changes.  Since this before-and-after detail will only ever be accessed as a whole, it's reasonable to make it one json bundle in a text field.

Archiving simply becomes a delete operation, as all the details are archived in the history.  This means, of course, that related data needs to be archived together, which is a good thing.  Furthermore, it's trivial to put a time limit on data retention; simply delete old journal entries.

My next API is going to rock.
