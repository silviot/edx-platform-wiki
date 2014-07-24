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

This code executes in a "jailed" environment to protect the servers.  The list of python packages available to this code is listed [here](https://github.com/edx/edx-platform/tree/master/requirements/edx-sandbox).  In addition, [python's standard libraries](https://docs.python.org/2/library/) are available.