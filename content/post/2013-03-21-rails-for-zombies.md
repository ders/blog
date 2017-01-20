+++
date = "2013-03-21T12:19:00+09:00"
title = "Rails for Zombies"

+++

I got introduced today to [an excellent Ruby on Rails tutorial](http://railsforzombies.org) with an entertaining zombie theme.  It covered a lot of the basic concepts, many of which I'd skipped over in my haste to dive into a real-live project.

Being who I am, I noticed a couple of inconsistencies between the tutorial videos and the exercises.  Actually, this only applies to the level 5 video; the rest seemed to be fine.

In the level 5 video at 2:10, the following match example is given:

```
match 'new_tweet' => "Tweets#new"
```

Note the uppercase T on Tweets.  This T is uppercase throughout the video.

However, when I tried to do the second exercise for this level, the following answer was rejected:

```
match 'undead' => "Zombies#undead"
```

The hints told me to do this:

```
match 'undead' => "zombies#undead"
```

with a lower case z, which was accepted.  Now I'm confused.  Do we need a capital letter here or not?

In the same video at 3:30, the following match example is given:

```
match 'all' => redirect('/tweets')
```

Note that there is no leading slash on 'all'; this format is consistent throughout the video.

However, when I tried to do the third exercise, the following answer was rejected:

```
match 'undead' => redirect('/zombies')
```

This time the hints told me to do this:

```
match '/undead' => redirect('/zombies')
```

Again, I'm confused. Do we need (or even want) a leading slash here?
