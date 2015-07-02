+++
date = "2013-02-22T10:41:00+09:00"
title = "Tip of the day"

+++

When running the Mongo shell for the first time, it is necessary to specify the port:

{{< highlight text >}}
$ mongo 127.0.0.1:27017
{{< /highlight >}}

Subsequently it is sufficient just to to say `mongo`.

Learned the hard way, and by reading [this answer](http://stackoverflow.com/a/8509918).
