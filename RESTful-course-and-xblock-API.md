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

## General syntax pattern and operations' semantics

## Specific API (subset)