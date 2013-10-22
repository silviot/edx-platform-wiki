The edx platform is moving to a RESTful service-oriented interface for most data and page access. This page will not attempt to define nor defend RESTful as there are many better places for that discussion (wikipedia and google are your friends here.)

The first step will be defining and implementing the RESTful interface. The second will be locating and isolating it properly in the architectural implementation so that the services can be easily scaled.

## What will it address?

The RESTful interface should make full CRUD access available for all xblocks, courses, and ancillary information (e.g., assets). Its default will be to consume and produce json representations of these, but the same url patterns should also support generating application appropriate html by setting the HTTP_ACCEPT to `text/html`.

## Authorization requirements

In a separate document (on the edx internal wiki), we debated and decided on authorization requirements for now. Of course, these are subject to revision. We have a short-term and a desired near-term set of requirements. The difference merely reflects the current limitation of our editing interfaces and representations.

Authorization differs by content type as follows.

### Course information

To retrieve course names, identities (urls), enrollment dates, and marketing information, in the short run we will make these wide open without requiring even authentication. Our near-term desire is to allow the course authoring team or their parent organization (anywhere up the org hierarchy) to control access as fully open, any authenticated user, any user in a whitelist of domains, user-specific whitelist (invitees or via unlock key), and/or dates (enrollment or other dates).

To create or delete courses requires special server permission now (being an edx staff member or being granted privileges by an edx staff member). In the near-term, we'd like to delegate the permission granting and revocation to the authorized institutions using our service. They'd each ask for and receive a set of organization designations (similar to domain names) which only the people they've authorized can use. We may, hopefully, allow each institution to create subdomains and delegate permissioning for course creation and deletion in those subdomains to whomever the institution wants (e.g., a department within a university such as UTCx.chem could control that subdomain).

To update (rename, change dates, change marketing info) course info requires instructor or staff access within that course. Studio provides a screen for managing this access within each course.

### xblocks

To retrieve an xblock (json model or html) in the short-term, requires authentication and enrollment (or equivalent course-specific privilege). Our near-term desire is to give the course staff the ability to set course-wide and then specific xblock privilege. A difficult issue with block-specific privileges has to do with the hierarchical nature of xblocks. We'll need to decide whether access to a child is the max of the privileges of itself and its ancestors or can it be more open than its ancestors (or is only its "content" scoped information so open but its "settings" and "children" are subject to the inherited max?).

The author set settings options would include public, authentication-required, whitelisted user domains, whitelisted users (including via course registration or unlock key), and whitelisted applications (e.g., for grading mechanisms). In all cases, the course team (instructors and staff) would have read access.

To create, delete, or update xblocks will require instructor or staff access in the course pointing to the xblock. (Note, we'll need to figure out access permissions for detached xblocks.)

### assets

Assets are similar to xblocks, however, Studio already has the ability to lock individual assets versus making them public. The default is public access unless locked, then it requires course registration. Assets are also not hierarchical; so, the ancestor permissioning question does not apply to them.

## General syntax pattern

The general syntax is a prefix indicating the type of information (not mime encoding but domain type) followed by the new locator serialized id representation (usually a course id with branch and block information.)

The url should avoid implying an action and strive to be as context independent as possible.

### Course information

A course always requires a course id and usually requires a branch id. See [Locators and older Locations](https://github.com/edx/edx-platform/wiki/Locators-and-older-Locations) for details. In the short term, the course id will be `org`.`course_number`.`run_name`. For example, `mitx.6.002x.t1_2013`. In the short term, the branch will either be `draft` or `published`. In the future, these can be almost any string with the course id hopefully expressing organization, department, major, course catalog id, and other hierarchically identifying information in addition to the run id.

The short-term url syntax will thus be something like `http://course/mitx.6.002x.t1_2014/branch/published` indicating that the sender wants to get course information for the given identified course.

Note, it's always legal to insert `edx://` before the course id but none of the url interpreters require it unless the type being addressed is unclear (i.e., it's not clear whether it's a context-relative xblock or a context-independent xblock definition). courses are always a context by definition; so, the `edx` tag never clarifies the semantics in its url. However, the above url could equivalently be written as `http://course/edx://mitx.6.002x.t1_2014/branch/published`

### xblock information

Most xblock access will be with respect to a course. Some, however, may be with respect to a specific structure version. Other access may be to only get the definition (context-independent content).

The course relative access with be using urls which merely add `/block/_blockId_` to the course url syntax. For example, `http://xblock/mitx.6.002x.t1_2014/branch/published/block/root`. As above, the caller may insert `edx://` before the course id with no adverse effects.

To get a specific version (note these are non-course run specific: i.e., they may be shared among course runs), the syntax is `/version/_guid_/block/_blockId_`; thus, for example,  `http://xblock/version/0123456789abcdef0123456789abcdef/block/root` which may return the same xblock as the course request above. As above, the caller may insert `edx://` before `version` with no adverse effects.

xblock definitions are never course relative and, for now, always use guids for ids. So, addressing them is via either a `defx://` tag or by the prefix of the url implying the type being a definition. Examples: `http://definition/0123456789abcdef0123456789abcdef', `http://definition/defx://0123456789abcdef0123456789abcdef'.

## operator and header semantics, data types

The request operator and headers should follow the wc3 standards. Briefly, use `GET` to retrieve data, use `POST` usually without a specific id but with a full payload and url prefix to create a new instance of some data, use `PUT` with a fully identified url to update an existing instance's definition, use `DELETE` with a full url to delete an existing instance.

The most important header attribute is the `ACCEPT` attribute. We will try to write `GET` handlers for both `application/json` and `text/html`. We will write `POST`, `PUT`, and `DELETE` handlers for `application/json`. We may add other types in the future.

## Specific API (subset)

As of October 2013 we're implementing handlers for `course`, `assets`, `checklists`, and `item`. `course` operates on the general course information as in the above section. `assets` operates on uploads using the course-relative syntax (not version relative). `checklists` operates on a specific course's checklist (a Studio course lifecycle work tracking system).

### `course`

A `GET` without any arguments should return all courses. A `POST` should create a course if the json payload is valid (not implemented yet).

Any method invocation with a fully specified course id, should do the relevant operation on the course. In the case of `GET`