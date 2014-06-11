A better set of tools around profiler viewing

A more full fledged profile viewer using http://urwid.org/

a tree view example:
http://urwid.org/examples/index.html#browse-py
which would be great for generating something like the thread profiles in https://rpm.newrelic.com/accounts/88178/key_transactions/8111/...
(with the cumulative numbers)

the pstats module already has a commandline functionality (python -m pstats <profile file>), but actually being able to drill down and see the tree is a pain

there are a lot of features that i want to add in my happy world. things like searching for methods, sorting in different ways, accumulating multiple profile files (pstats the lib supports that), profile diffs, automatically flagging things that do i/o like db and file operations, etc.
but just starting with the ability to view a cumulative run view using the pstats lib and urwid in a tree 
would be great

Other things to use as reference: https://github.com/nedbat/memsee

http://stefaanlippens.net/python_profiling_with_pstats_interactive_mode (what's already built into the stdlib pstats lib for interactive mode -- just background info, but would be a useful way to test that your system is interpreting data correctly)

http://pymotw.com/2/profile/ (explains some of the stats in more detail -- note ncalls recursive vs not explanation)

http://zameermanji.com/blog/2012/6/30/undocumented-cprofile-fe... (undocumented cProfile features -- probably not of immediate interest because a lot of this stuff doesn't get serialized into the pstats file by default, but interesting)

pstats source code: http://hg.python.org/cpython/file/f89216059edf/Lib/pstats.py

and of course, lots of profiles to try out:

https://edx-wiki.atlassian.net/wiki/display/LMS/Opaque+Keys+Profile...

fwiw, almost everyone uses cProfile to generate pstats files. i'm not sure if profile produces slightly different data, but i think it's safe to ignore for the first pass. all the profiles nimisha, cale, and i posted were generated with cProfile

urwid's a little light on docs for some features, so it might be easiest to start with their browse.py example and modify it to suit your needs, rather than building one from scratch. though i haven't looked into that hardly at all.

https://github.com/wardi/urwid/blob/master/examples/brows...

http://urwid.org/