---
layout: post
title: "Tracing Riak Nodes with DBG"
date: 2017-03-22
---

I'm not sure on the best approach for tracing a Riak node, but I know that if
you're connecting to the shell by any other means than riak console, your
tracing isn't going to work as expected.

The reason for this is that you're actually connecting to this node using the
-remsh switch. This causes Erlang to connect to the runtime using a hidden node.
Any processes that are started within that shell are actually started on the
primary runtime, not in the local.

For this reason, when we try to run dbg after connecting to a riak node. The
tracer that we start up is running on the primary console, thus, it will output
trace  messages to the stdout of that shell.

To get around this problem, I figured out that I could connect to the local 
shell of the hidden node, and then run the tracer from there. There's a convenient 
function for running a process tracer on a remote node, too. This can be
achieved by doing:

```bash
riak attach
```
Now we're on the Riak node (actually connected with a remsh)
~~~
^G (Display the user switch menu)
~~~
~~~
s (Start a new local shell)
~~~
~~~
c (Connect to the new shell)
~~~
Now we're connected to the actuall hidden node. We can start dbg here and then
run a process tracer on the Riak node, which will give us the tracer output.
~~~
dbg:start()
~~~
~~~
dbg:tracer()
~~~
~~~
dbg:n('riak@127.0.0.1') (Start a process tracer on a remote node)
~~~
~~~
dbg:tp(lists, seq, x)
~~~
~~~
dbg:p(all, c)
~~~
Sit back with a beer and watch the trace output roll in...

It cost me some hair to work that out. A colleague at work recently had the same
issue, which led me to write this post. Hopefully it can help to save someon
else some time and hair.

Enjoy!
