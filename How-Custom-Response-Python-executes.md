`<customresponse />` elements in "Advanced Problems" can be graded using python provided in `<script type="loncapa/python">` tags, such as the example below which basically marks all responses as correct

```python
<script type="loncapa/python">
import re
  
def short_response(expect, ans):
  response = re.search('', ans)
  if response:
  	return 1
  else:
  	return 0
</script>
```

But what kind of python code can be specified within these script tags?  For example, what python libraries are available to this python code?

This code executes in a jailed environment to protect the servers.  (Links for the software performing this enforcement, both of which have detailed readmes: [`codejail repo`](https://github.com/edx/codejail) [`safe_exec in edx-platform/common/lib/capa`](https://github.com/edx/edx-platform/tree/master/common/lib/capa/capa/safe_exec).)   First, this environment has a restricted python environment.  The list of python packages available to this code is listed in the edx platform repo at [requirements/edx-sandbox](https://github.com/edx/edx-platform/tree/master/requirements/edx-sandbox).  In addition, [python's standard libraries](https://docs.python.org/2/library/) are available.

Also, python code specified in these script tags are subject to OS level restrictions provided by [Ubuntu's apparmor package](https://wiki.ubuntu.com/AppArmor) that, among other protections, limit execution time and prevent file and network access.