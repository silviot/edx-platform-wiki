As part of the Modulestore Data Structure refactoring, we've refactored the old Locations to a new set of Locators. This page explicates the url syntax for courses and blocks (nee, modules) covering naming constraints and recommendations.

## Old style Locations

Before explaining the new, I will briefly discuss the old. Parentheses denote optional elements.

* `i4x://org/courseid/category/blockid(@draft)`
* `['i4x', org, courseid, category, blockid, ('draft')]`

The old syntax assumed that the organization name (`org`) and a course identifier (`courseid`) suffice to uniquely identify any course offering and restricted course identity to those 2 fields. Of course, this assumption failed for courses with multiple runs. The old syntax exposed `category` as part of the name where category is the keyword used to determine which XBlock class to instantiate. It then appended a unique id (`blockid`) for the module within the context of the course id and category. In most cases this unique id was unique over the whole course or even over all courses.

The old syntax provided the ability to optionally append '@draft' to specify that the url wanted to access the draft version of a module. It never supported any other versions.

## Locators

Locators allow addressing of courses, course-like structures not necessarily used by courses, blocks within courses, and definitions for blocks. Because of this flexibility, they're more complex than Locations. The addressing of a block within a course does the same function as Locations; so, I'll limit this discussion to those. Even these are more flexible than Locations and allow for more optional elements (demarcated with parentheses). The optional version can be either a named tag or a specific version id (akin to a commit id).

`(edx://)structured_course_id(/branch/branch_name)(/version/version_id)(/block/blockid)`

All characters should be legal characters for urls. The system will not rewrite any of the ids by replacing spaces with underscores or such but will instead just pass the strings through urlencode to escape any included illegal chars. Also, all fields will be case insensitive; so, do not use case to distinguish ids nor tags.

### structured_course_id

A structured_course_id can be any urlencodable sequence of characters which uniquely identifies the course. Neither Locators nor the mongostore impose any particular syntax nor restrictions other than uniqueness on these; however, we strongly suggest and may possibly require that they follow a specific syntax and pattern to enable future smarter dashboards, delegated governance, and data aggregation.

We strongly recommend and may require in the future that course ids follow this convention: that they are a dot separated scoping list identifying organizations and then optionally course families followed by the organization's course id and optionally a run id. We decided the name should not strictly follow java-like packaging with the inclusion of 'edu' or 'com' as the first element. Here are some examples starting from simple to more encompassing:

* `utc.orgch2` – where utc is the org and orgch2 is the catalog identifier of the course
* `utc.chem.orgch2` – where chem is the department
* `utc.chem.organic.advanced_intro` – where organic designates a course family and advanced_intro is more descriptive than the catalog id
* `utc.chem.organic.advanced_intro.2013_SOND` – where 2013_SOND designates a specific run of the course.

This type of structured identifier allows us to group courses in our catalog by school or department across schools and other such means. It also allows us to aggregate data by course family, department, or school. Finally, and perhaps more importantly to self-sufficiency, it allows us to grant organizations (schools, companies) governance over their namespace where they can decide what can use their org id and, thus, who can access their org id. It further gives those organizations the ability to delegate subnamespaces to departments, etc.

### branch_name

The branch_name is optional. Each course will have a default version. By default, Studio uses `draft` and `published`. Version tags typically distinguish draft versus published but allow for multiple variants to exist and continue being extended at the same time. The persistence layer provides methods for publishing from one version to another which is akin to cherry-picking changes from one branch to another. The version tag points to the current head of its respective branch and can be reverted to an earlier or pointed to a completely different structure.

The system makes no requirement on version tags other than they be urlencoded. Different organization may establish their own naming conventions such as using 'draft' or 'live' for draft versus published.

* `edu.utc.chem.organic.advanced_intro/branch/draft`
* `edu.utc.chem.organic.advanced_intro/branch/live`

### version_id

The optional version id points to how a course looked at a specific point of time. It actually points to a whole course-like structure and doesn't require any other information for identifying the course structure. Each change to a course causes a new version. Version ids are UUIDs and look like long hexidecimal strings. If you modify a course via its version_id, no version tag will not move to the resulting new version. There are methods for pointing version tags to specific versions, however.

* `edu.utc.chem.organic.advanced_intro/version/519665f6223ebd6980884f2b`
* `version/519665f6223ebd6980884f2b`

It is possible to provide both the version tag and id. In this case, if the tag does not match the current head of the version tag, you get a version conflict error. This is how Studio stops simultaneous edits of the same block.

### blockid

The blockid uniquely identifies a block within a course version. The current code does not guarantee that the same blockid in 2 different course versions identify the same semantic block because it opts for more human readable ids and the only real way to guarantee uniqueness over variants is with UUIDs.

The current implementation names blocks by their categories in serial order of creation. So, the first chapter created is 'chapter1', the 7th is 'chapter7'.

`edu.utc.chem.organic.advanced_intro/branch/live/block/chapter1`
