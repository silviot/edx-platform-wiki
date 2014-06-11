but what i really wanted to play with was making a better set of tools around profiler viewing.
cause 1) getting RunSnakeRun to go on a mac is horrible; and 2) RunSnakeRun isn't really that great
so i really wanted to build a more full fledged profile viewer using http://urwid.org/
in particular, there's a tree view example:
http://urwid.org/examples/index.html#browse-py
which would be great for generating something like the thread profiles in https://rpm.newrelic.com/accounts/88178/key_transactions/8111/...
(with the cumulative numbers)
the pstats module already has a commandline functionality (python -m pstats <profile file>), but actually being able to drill down and see the tree is a pain
there are a lot of features that i want to add in my happy world. things like searching for methods, sorting in different ways, accumulating multiple profile files (pstats the lib supports that), profile diffs, automatically flagging things that do i/o like db and file operations, etc.
but just starting with the ability to view a cumulative run view using the pstats lib and urwid in a tree would be great