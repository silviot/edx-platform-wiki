@sefk asked @jrbl to put together a checklist that Stanford's OpenEdX team could use to make the process of QA'ing pull requests more straightforward. We want to eliminate thinking about "What do I need to check?" to focus thought on "How well did they do X?". Note that this is subtly different from the Code Review, which is focused more on engineering issues, like performance trade-offs and overall design. To be sure, there's a lot of overlap, but a moment's reflection will make clear that one can have well-engineered code that fails the quality standards, and high-quality code that doesn't do anything useful.

# Before you Review #
* Read the [[Python Guidelines]]. Especially follow the links to Pep8 and Pep257.
* Read [Writing and Running Tests](https://github.com/edx/edx-platform/blob/master/doc/testing.md)
* Read [Internationalization/Localization](https://edx-wiki.atlassian.net/wiki/pages/viewpage.action?pageId=12517501&src=search)
* Read [Strunk & White](https://en.wikipedia.org/wiki/The_Elements_of_Style)

# The QA Checklist #
* Quiz the authors (face to face if possible) about how things work and how it's been manually tested. Think about functionality corner cases (configuration left undefined, very high usage, very low usage, etc.), and brainstorm likely design mistakes. Generally convinced yourself of the authors' conscientiousness.
* Is test coverage over 95%? (If not, does that make sense?)
* How is test style and coverage, conceptually? (e.g., Is there a clear mapping between tests and new/edited code? Is new code covered by at least a smoke test? Are edits?  If a bugfix is involved, do the tests ensure that the pre-fixed code will fail, and the post-fixed code will pass?)
* Check pylint and pep8 violations - they should not increase over master. With a recent rebase, this should be verifiable by running ```rake quality```
* Are strings output to the user in LMS and CMS internationalized? Do you see the ```_()``` being used everywhere it makes sense?
* Skim the code for structure; does it have a nice shape? (e.g., nontrivial work will probably span more than one file, functions shouldn't be too long, variable names should be descriptive, etc...)
* Run the tests!
* Read the docstrings. Are there any? Are they accurate? Do new classes/modules have good descriptive strings? Are they free from grammar and spelling mistakes?
* Read the commit history. Do the commits fit together in a nice way? Most changes can probably be squashed into one big commit, or a few commits whose commit messages "tell a story".  See "A note about squashing" below.
* How do the commit messages read? Check grammar and spelling, but also formatting and completeness. See "A note about commit messages" below.
* Have AUTHORS and CHANGELOG been updated?

## A note about squashing ##
If you're at all intimidated by history management, ask around! There are several people around who have strong git-fu that can help you. 

There are many ways to elide history, but in many cases a rebase-and-squash is adequate. If the branch is long-lived with many merges (rather than rebases) from master in its history, the least-effort thing may be to make a branch that
[throws history away but keeps the changes](http://stackoverflow.com/questions/1464642/git-merge-squash-repeatedly). One could do that multiple times, and cherry-pick the later squashes onto the
earlier ones to create a series of commits that tell a story, but this is untried, and usually it will be sufficient to rewrite history to one-big-commit. If rewriting commits from more than one author, check out the example commit message below.

## A note about commit messages ##
Commits should have a 50 character summary line, a blank line, and a body in one or more paragraphs using complete sentences that describe what the code does generally, in what contexts, what configuration variables it needs set, any gotchas for people who *don't* want to use it, and which document environmental assumptions. Ideally, next week's release master should be able to read only this commit message to decide whether to release this change and to find out how to release it. Or at least, the commit message should point them to a doc or wiki where they can find out more; the point is you don't want them having to come and find you next week.  Also when squashing commits from multiple people, secondary and tertiary contributors can be recorded with a linebreak and one or more "Co-authored-by" lines, e.g.:
```
Sample commit makes everything into rock candy

This commit does stuff. It's really sweet, and it totally rocks.  Nothing needs
to be done to deploy, it's enabled by default. If you want to continue to use
the old, bland behavior, just set IHATECANDY=True in asw.py.

Co-authored-by: Willy Wonka <wonkawonka@example.com>
Co-authored-by: Rocky Balboa <rocky.pugnacious@example.com>
```