(Originally written by Sef (<sef@stanford.edu>), transcribed by Ned)

# Five Ways to Extend edX

I can think of five ways that someone could extend edX.  Here they are in order of difficulty:
1. **jsinput** -- Create a "Custom JavaScript Grade and Display" component, and then provide JS with getState(), setState(), and getGrade() methods.
2. **LTI** -- edX supports LTI 1.1 now, LTI 2.0 in development
3. **custom grader** -- Code can be run on an external server to do arbitrary work to grade problems.
4. **XBlock**
5. **hack on core code**

Here’s my initial attempt at a grid to summarize , but I’m sure there are more rows that we’d want to consider:

<table>
<tr>
<th>&nbsp;</th>
<th>JSInput</th>
<th>LTI</th>
<th>Grader</th>
<th>XBlock</th>
<th>Core Mod</th>
</tr>
<tr>
<td>Cost</td>
<td>Low</td>
<td>Low</td>
<td>Med</td>
<td>Med</td>
<td>High</td>
</tr>
<tr>
<td>Language</td>
<td>JS</td>
<td>any</td>
<td>any</td>
<td>Python</td>
<td>Python</td>
</tr>
<tr>
<td>Need dev environment</td>
<td>no</td>
<td>no</td>
<td>yes</td>
<td>yes</td>
<td>yes</td>
</tr>
<tr>
<td>Self-host component</td>
<td>no</td>
<td>yes</td>
<td>yes</td>
<td>no</td>
<td>no</td>
</tr>
<tr>
<td>Need edX involvement</td>
<td>no</td>
<td>no</td>
<td>yes</td>
<td>yes</td>
<td>yes</td>
</tr>
<tr>
<td>Clean UI integration</td>
<td>yes</td>
<td>no [1]</td>
<td>yes</td>
<td>yes</td>
<td>yes</td>
</tr>
<tr>
<td>Mobile friendly</td>
<td>no</td>
<td>maybe</td>
<td>yes</td>
<td>yes</td>
<td>yes</td>
</tr>
<tr>
<td>Server Side Grading</td>
<td>no</td>
<td>yes</td>
<td>yes</td>
<td>yes</td>
<td>yes</td>
</tr>
<tr>
<td>Usage Data</td>
<td>no [2]</td>
<td>no</td>
<td>limited</td>
<td>yes (?)</td>
<td>yes</td>
</tr>
</table>


Notes:
* [1] Only LTI components delivered via https can be iframed in  Many are served over http only.  And even then they usually have their own look and feel.  For example, Piazza can be iFramed in, but has its own navigation elements and their color scheme (see <http://networking.class.stanford.edu/> for an example).
* [2] JSInput really only exposes the getState, putState, getGrade methods.  But is there any reason why we can’t also document / publish the tracking endpoint (/events/user_track I believe)

