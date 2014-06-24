Several groups are currently working on REST APIs for edx-platform (Analytics,
Solutions, Mobile), and we need conventions to present a consistent interface to
API clients.

An incomplete list of challenges (leaving authn/authz aside for now):

1. Canonical entity representation (e.g. Course, User, Enrollment)
2. Versioning
3. Enabling third party additions
4. Discovery

### Example Using a Canoncial Entity

There are certain entities that are such a core concept to edX that they are
used everywhere. That being said, it is often difficult to agree on what a
canonical representation of these entities even is, and different parts of
the system may have conflicting ideas of what constitutes a Course, for 
instance. This problem is compounded by the need to enable the creation and
easy discovery of new APIs, many of which will not be developed by third
parties.

The proposal here attempts to sidestep the problem by presenting an extremely
minimal base representation, exposing most functionality via a pluggable
interface that third parties can register for. As an example:

```JavaScript
// GET /api/edx.core/courses/edX+Open_DemoX+edx_demo_course
{
    "course_id": "edX+Open_DemoX+edx_demo_course",
    "name": "edX Demonstration Course", 
    "_links": {
        "edx.analytics": {
            "enrollment": {
                "url": "https://analytics-api.edx.org/courses/edX+Open_DemoX+edx_demo_course/enrollment"
                "versions": ["1", "dev"]
            },
            "demographics": {
                "url": "https://analytics-api.edx.org/courses/edX+Open_DemoX+edx_demo_course/demographics"
                "versions": ["dev"]
            }
        },
        "edx.content": {
            "xblock": {
                "url": "https://api.edx.org/api/edx.content/xblocks/edX+Open_DemoX+edx_demo_course"
                "versions": ["2", "3", "dev"]
            }
            "authors": {
                "url": "https://api.edx.org/api/edx.content/courses/edX+Open_DemoX+edx_demo_course/authors"
                "versions": ["2", "dev"]
            }
        },
        "edx.videos": {
            "video_listing": {
                "url": "https://api.edx.org/api/edx.video/video_listing/courses/edX+Open_DemoX+edx_demo_course"
                "versions": ["dev"]
            }
        }
    },
    "_version": "1",
    "_versions": ["1"]
}
```

The idea would be that installed packages could define any number of resources
in different namespaces. The namespaces would represent logical groupings, and
may cross app boundaries (say related apps in a repo). The basic guidelines:

1. Wherever feasible, items linked to in "resources" should include back links
   to the canonical entity.
2. These would often be Django apps, but could also be very thin config pieces
   that do nothing more than point to services living on a different server.
3. If there are versions specified for a given URL, it means that it will obey
   the convention of passing a `?v={version}` querystring parameter. Without
   an explicit version, apps should default to the latest. This does require a
   little more intelligence on the part of the client, but it makes things much
   more compact. There is the problem of moving the URLs -- this proposal would
   requires that either a new resource entry is made or that the URL pointed to
   would itself be responsible for doing the redirection.
4. These URLs have to be super-cheap to generate. Because there are going to be
   many things hanging off of it, it should just be a dictionary lookup -- no
   database or network access to determine whether or not a resource URL will
   show up. This means that sometimes you might follow a link straight into a
   403 error and have to account for that.
5. The URL for the canonical entities also respect a version `?v={version}` param,
   but that version only applies to the minimal core data exposed. It does not
   apply to the resource list, which can change at any time when new services
   are added. 
6. EdX reserves the use of the "edx." prefix for resource groups, but third
   parties can register their own. The exact mechanism for how we do this hasn't
   been determined yet.

### High Level Discovery

Each API would also define a top level entry point that would be shown when a
request is made to `edx-platform` on `/api`
