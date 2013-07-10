@sarina and @peter-fogg added support for code quality checking in `diff-cover`.

The branch: [https://github.com/edx/diff-cover/tree/quality-violations](https://github.com/edx/diff-cover/tree/quality-violations)

The Pull Request [https://github.com/edx/diff-cover/pull/22](https://github.com/edx/diff-cover/pull/22)

Usage of `diff-quality` is similar to that of `diff-cover` (see the README for more details). The report format is modified to include line numbers and error messages from the checker. Currently only `pylint` and `pep8` are supported; however, it should be quite easy to add support for additional style checkers (`jslint`? `coffeelint`?).

Here is how you use the pep8 and pylint quality violations reporters for a console output:

![pep8 quality violations in console](http://d1zjcuqflbd5k.cloudfront.net/files/tmp_4788fc90d6467e212bc4d0035f652c06/O9Jq?response-content-disposition=inline;%20filename=Screen%20Shot%202013-07-10%20at%201.04.29%20PM.png;%20filename*=UTF-8%27%27Screen%20Shot%202013-07-10%20at%201.04.29%20PM.png&Expires=1373494609&Signature=D0ZFsEsY67JPPEcK8oIev9kWBCvgLCBfVKjCAjzlic8SbLQMga62hmio0A7DbLuzOZLrDFa9vgINJ7QuP-EspzafeEkqTst-0RnwUGFPaxLG2KiD-TrcnlAqXzf73Hw48JOM~e50WjtmJWlqZo1tYtMo28JPlr4j1YgnAW9PDXU_&Key-Pair-Id=APKAJTEIOJM3LSMN33SA)

![pylint quality violations in console](http://d1zjcuqflbd5k.cloudfront.net/files/acc_153389/m7lV?response-content-disposition=inline;%20filename=Screen%20Shot%202013-07-10%20at%206.07.31%20PM.png;%20filename*=UTF-8%27%27Screen%20Shot%202013-07-10%20at%206.07.31%20PM.png&Expires=1373494610&Signature=IbBh9C-v0hklgD6cbkO2BZq6ySJzURfhB-1mA7gv~pO4s2w3coWnpKEUw5lrjzYU2pr6EdvqxEP0ThHUX1HH4OH5LLlLrjc3Ut~pV-PGfbFDwTS9M~dKgyNXZ88QS6Y6JKJFTT2Y5ftgr4y0s2cVTLl-jgfQiTfKkWO2JOoGs~8_&Key-Pair-Id=APKAJTEIOJM3LSMN33SA)

Adding `--html-report=path/to/report.html` to the end of the command will generate an HTML report at the specified location. The html reports look something like this:

![pep8 html report](http://d1zjcuqflbd5k.cloudfront.net/files/acc_153389/OP47?response-content-disposition=inline;%20filename=Screen%20Shot%202013-07-10%20at%201.04.44%20PM.png;%20filename*=UTF-8%27%27Screen%20Shot%202013-07-10%20at%201.04.44%20PM.png&Expires=1373494554&Signature=C~gmwrFvJpK7fDOxbwAoWx6DgdPN02rL8u-4~IQACmBee9He31itb11jMfG10Kouo4I2mXrIj8f0o3RGBt27wyYx3OnSTOVNv5Gtlcku8pRMiVyY~~0DXI9vNrSeycdQwHZHLhHp29CSAb3-DNTseDnKJN~tLV0q8ZpfhqPMWgQ_&Key-Pair-Id=APKAJTEIOJM3LSMN33SA)

![pylint html report](http://d1zjcuqflbd5k.cloudfront.net/files/acc_153389/wBQT?response-content-disposition=inline;%20filename=Screen%20Shot%202013-07-10%20at%206.07.38%20PM.png;%20filename*=UTF-8%27%27Screen%20Shot%202013-07-10%20at%206.07.38%20PM.png&Expires=1373494558&Signature=fGgKQU9WjHvnkVF-mXdxIBxpSN~2WIpsjlNVbu1EdbOA36KSm3BYsbX2~M1tueKMaa8mnrPQQjXUdATghc1SCDvXWBkaLmSzae0xtR2Kfpe7ovfq0nYXbMVMkqcAG6vrz8Sc6jKviL~1hP4hBeh7FeMl~i6FfqDKSQB0XpmLvV8_&Key-Pair-Id=APKAJTEIOJM3LSMN33SA)