Here's a cheat sheet for those of us updating Studio to support opaque keys.

**branch name:** opaque-keys
**PR:** https://github.com/edx/edx-platform/pull/2905
**commit using:** git commit --fixup 2eca90226ba6282499e0a20a76fcec91bbe97027
**STUD-452:** https://edx-wiki.atlassian.net/browse/STUD-452

1. Change Reverse calls
course_url = reverse(
     'contentstore.views.course_handler',
     kwargs={'course_key_string': unicode(course.id)}
)

2. 
