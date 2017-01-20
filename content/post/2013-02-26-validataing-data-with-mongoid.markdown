+++
date = "2013-02-26T15:57:00+09:00"
title = "Validating data with Mongoid"

+++

I've been working with [Mongoid](http://mongoid.org/en/mongoid/), which is an object-document-mapper for [MongoDB](http://mongodb.org/) written in Ruby.

Mongo organizes data into collections of documents, just as relational databases such as SQL organize data into tables of records.  Reading and writing of
documents is done via named classes, one for each collection.

The named class for each collection includes `Mongoid::Document` to get the database interface methods such as `.where`, `.new`, and `.save`. It also defines the data fields and any custom data handlers.

One very useful feature is the availability of automatic validators which check the format and integrity of your data before allowing it to be saved.  There's a myriad of options, and they are [not very well explained](http://mongoid.org/en/mongoid/docs/validation.html) in the documentation.

Since the data validators are shared with [Active Model](http://api.rubyonrails.org/classes/ActiveModel.html), I decided to look for some help there and found [this pretty good description](http://apidock.com/rails/ActiveModel/Validations/ClassMethods/validates) of what kind of validation could be done.
Unfortunately, it wasn't clear anywhere how to actually use the validators once they're defined.

After a bit of hair-pulling, I discovered it's actually quite simple.

Let's define a minimal class `Iqscore`.  (The name of the collection will be `iqscores`; this is a weird behavior of Mongoid whereby class names must be singular and Mongoid will pluralize them for you when naming the collection.)

```
require "mongoid"

class Iqscore

  include Mongoid::Document

  field :kid, :type => String
  field :iq,  :type => Integer

  validates :kid, :presence => true, :uniqueness => true
  validates :iq, :numericality => true

end
```

Mongoid provides a `valid?` method on Iqscore objects.  Valid? tells us whether or not the criteria in the `validates` declarations are met.

```
1.9.3p385 :008 > x = Iqscore.new({kid: "George", iq: 70})
 => #<Iqscore _id: 512c72d5352420234d000003, kid: "George", iq: 70>
1.9.3p385 :009 > x.valid?
 => true
```

```
1.9.3p385 :010 > y = Iqscore.new({kid: "Bill", iq: "unknown"})
 => #<Iqscore _id: 512c736e352420234d000004, kid: "Bill", iq: 0>
1.9.3p385 :011 > y.valid?
 => false
```

If it doesn't validate, we can see what's wrong by looking at the `errors` property.  In this case it tells us that `iq` is not a number (and it should be).  Note that the message "is not a number" is in an array, as it's possible for there to be multiple messages for a single field.

```
1.9.3p385 :012 > y.errors
 => #<ActiveModel::Errors:0x98a31cc @base=#<Iqscore _id: 512c736e352420234d000004, kid: "Bill", iq: 0>, @messages={:iq=>["is not a number"]}>
```

The `valid?` method is called automatically before any save operation (e.g. `save` or `create`), and if it returns false, then the save is not done.  Both `save` and `create` return true or false to indicate whether the save was done or not.

```
1.9.3p385 :013 > x.save
 => true
1.9.3p385 :014 > y.save
 => false
```

At this point George is in our database, but Bill isn't.
