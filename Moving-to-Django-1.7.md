<table>
<tbody>
<tr>
<th>Django Version</th>
<th>Necessary Work</th>
<th>Immediate Benefits</th>
<th>Additional Benefits (requires work)</th></tr>
<tr>
<td>1.5</td>
<td>@clintonb has done <a href="https://github.com/edx/edx-platform/tree/clintonb/django-upgrade">some work on the 1.5 upgrade</a>, however a few tests are broken.</td>
<td>
<ul>
<li>Password-length DOS protection</li></ul></td>
<td>
<ul>
<li>Use configurable user model to remove first/last name fields</li>
<li>Field subset saves to only update grades on courseware StudentModel</li>
<li>Easier logging of failed logins (new django signal)</li>
<li>Multi-column indexes</li></ul></td></tr>
<tr>
<td>1.6</td>
<td>
<ul>
<li>Remove <code>TransactionMiddleware</code></li>
<li>Enable <code>ATOMIC_REQUESTS</code></li>
<li>Replace use of<br />
<ul>
<li><code>autocommit</code></li>
<li><code>commit_on_success</code></li>
<li><code>commit_manually</code></li></ul></li>
<li>Wrap uses of <code>select_for_update</code> in <code>atomic</code></li>
<li>Repeatable read, and consistency between successive reads?</li>
<li>Check for <code>BooleanFields</code> with no explicit default</li>
<li><code>urlquote</code> in <code>reverse</code>?</li>
<li>Non-string session keys?</li>
<li><code>ModelForms</code> with no field list set?</li></ul></td>
<td>
<ul>
<li>Persistent DB connections</li>
<li>Faster <code>Model.save()</code></li></ul></td>
<td><ul>
<li>Safer session serialization</li></ul></td></tr>
<tr>
<td colspan="1">1.7</td>
<td colspan="1">
<ul>
<li>Remove use of <code>get_cache</code></li></ul></td>
<td colspan="1">
<ul>
<li>Easier DB routing for migrations (i.e. if edx-ora2 wanted to use its own DBs)</li></ul></td>
<td colspan="1">
<ul>
<li>New, better schema migration framework</li>
<li>Built-in app loading startup (can remove custom <code>startup</code> code)</li>
<li>Update_or_create for writing grades to CSM</li></ul></td></tr></tbody></table>