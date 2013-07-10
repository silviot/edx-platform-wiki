@sarina and @peter-fogg added support for code quality checking in `diff-cover`.

The branch: [https://github.com/edx/diff-cover/tree/quality-violations](https://github.com/edx/diff-cover/tree/quality-violations)

The Pull Request [https://github.com/edx/diff-cover/pull/22](https://github.com/edx/diff-cover/pull/22)

Usage of `diff-quality` is similar to that of `diff-cover` (see the README for more details). The report format is modified to include line numbers and error messages from the checker. Currently only `pylint` and `pep8` are supported; however, it should be quite easy to add support for additional style checkers (`jslint`? `coffeelint`?).

Here is how you use the pep8 and pylint quality violations reporters for a console output:

![pep8 quality violations in console](http://d1zjcuqflbd5k.cloudfront.net/files/tmp_4788fc90d6467e212bc4d0035f652c06/O9Jq?response-content-disposition=inline;%20filename=Screen%20Shot%202013-07-10%20at%201.04.29%20PM.png;%20filename*=UTF-8%27%27Screen%20Shot%202013-07-10%20at%201.04.29%20PM.png&Expires=1373494414&Signature=U5keugTwr6TaIhN-hu26boedZyl8630hJFDX4tCqYAB8UvxB51RLlG12muaRhntHtOm5dkdqkcbH4FNO3WkzoCziu5TNzyKwmFx7EUj6Fs-3zMb72SYNgo-Fuix9zzNhLepAHnWsMj5a7cCjGLXXjIvXHBktO6QB8WpXzTOylCA_&Key-Pair-Id=APKAJTEIOJM3LSMN33SA)

![pylint quality violations in console](http://d1zjcuqflbd5k.cloudfront.net/files/acc_153389/m7lV?response-content-disposition=inline;%20filename=Screen%20Shot%202013-07-10%20at%206.07.31%20PM.png;%20filename*=UTF-8%27%27Screen%20Shot%202013-07-10%20at%206.07.31%20PM.png&Expires=1373494478&Signature=H5r8p2GZAJA81HV2xGfpKmTT7lEH6jPxIRonO5YuG103dvDNDy7b-6kYoHaJe~u0ieLzgsIMSMg0Db56UcBvzT8ZNvSw9eYVucQ7y3~~TIw5bmPY6Q6CC93XVzbWkJk3mYHus4QdgqGOCqHUMpDQqitNHgOtcpTOZeJoULVI01M_&Key-Pair-Id=APKAJTEIOJM3LSMN33SA)

Adding `--html-report=path/to/report.html` to the end of the command will generate an HTML report at the specified location. The html reports look something like this:

![pep8 html report](http://d1zjcuqflbd5k.cloudfront.net/files/acc_153389/OP47?response-content-disposition=inline;%20filename=Screen%20Shot%202013-07-10%20at%201.04.44%20PM.png;%20filename*=UTF-8%27%27Screen%20Shot%202013-07-10%20at%201.04.44%20PM.png&Expires=1373494554&Signature=C~gmwrFvJpK7fDOxbwAoWx6DgdPN02rL8u-4~IQACmBee9He31itb11jMfG10Kouo4I2mXrIj8f0o3RGBt27wyYx3OnSTOVNv5Gtlcku8pRMiVyY~~0DXI9vNrSeycdQwHZHLhHp29CSAb3-DNTseDnKJN~tLV0q8ZpfhqPMWgQ_&Key-Pair-Id=APKAJTEIOJM3LSMN33SA)

![pylint html report](http://d1zjcuqflbd5k.cloudfront.net/files/acc_153389/wBQT?response-content-disposition=inline;%20filename=Screen%20Shot%202013-07-10%20at%206.07.38%20PM.png;%20filename*=UTF-8%27%27Screen%20Shot%202013-07-10%20at%206.07.38%20PM.png&Expires=1373494558&Signature=fGgKQU9WjHvnkVF-mXdxIBxpSN~2WIpsjlNVbu1EdbOA36KSm3BYsbX2~M1tueKMaa8mnrPQQjXUdATghc1SCDvXWBkaLmSzae0xtR2Kfpe7ovfq0nYXbMVMkqcAG6vrz8Sc6jKviL~1hP4hBeh7FeMl~i6FfqDKSQB0XpmLvV8_&Key-Pair-Id=APKAJTEIOJM3LSMN33SA)