Print Yourself Out
===

March 14, 2016

Original question from [>>/g/53463633](https://archive.rebeccablacktech.com/g/thread/S53463633).

> Write a program that prints itself.

Calm down with the jokes, no `print "itself"`s. Surprisingly, its pretty simple with bash, since they have a [special parameter to refer to itself](https://bash.cyberciti.biz/guide/$0).

```sh
#!/bin/sh
cat $0
```

There were no Python solutions, and since I am more or less studying Python, I figure I might as well give it a shot. The first (classical) approach I had was to look for the current script path, which lead me to `os.path.abspath(__file__)`. Curious about `__file__`, I tried printing it, and alas, its the script's filename. From there, its just a few more steps.

```py
#!/usr/bin/env python
# -*- coding: utf-8 -*-

with open(__file__) as f:
	print "".join(f.readlines())
```

Looks like Python had a variable that refers to itself as well. I think PHP has something similar too (`__FILE__` if I was right), so the logic is more or less similar to Python.
