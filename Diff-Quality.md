@sarina and @peter-fogg added support for code quality checking in `diff-cover`.

The branch: [https://github.com/edx/diff-cover/tree/quality-violations](https://github.com/edx/diff-cover/tree/quality-violations)

The Pull Request [https://github.com/edx/diff-cover/pull/22](https://github.com/edx/diff-cover/pull/22)

Usage of `diff-quality` is similar to that of `diff-cover` (see the README for more details). The report format is modified to include line numbers and error messages from the checker. Currently only `pylint` and `pep8` are supported; however, it should be quite easy to add support for additional style checkers (`jslint`? `coffeelint`?).

