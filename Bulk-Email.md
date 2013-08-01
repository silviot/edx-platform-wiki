<p align=center>
THIS IS A DRAFT DOCUMENT OF A FEATURE IN DEVELOPMENT.<br>
IT HASN'T BEEN PULLED INTO THE EDX-CORE YET.
</p>


## Overview

Course staff can now compose emails and send them to course staff.  The
key features are:

* Ability to compose HTML messages in the site.
 
* MIME multipart mails are sent
 
* Ability to efficiently to send to very large classes by fanning out
  jobs to worker machines over a job queue (celery).

* Mails are CAN-SPAM compliant: messages automatically have an optout link in the footer that directs them to the dashboard where users can set email settings for the course.

Credit goes to Jason Bau (<jbau@stanford.edu>) for developing the
feature in Class2Go.  Students Kevin Luo (<kevluo@stanford.edu>) and Akshay
Jaggadish <akshayj@berkeley.edu> took that code and ported to OpenEdX over Summer
2013.


## Feature Description

Instructors access the feature through the instructor dashboard by clicking the Email link.
This is toggled by the feature flag ENABLE_INSTRUCTOR_EMAIL.
![Creating an email](image/bulkemail-editor.png)

## Backend

The djangoapp bulk_email handles the email-sending action by getting the recipient
list and batching the emails to different celery tasks to do the actual sending. The
number of emails each task sends is configurable with EMAILS_PER_TASK. It requires
the lynx package to convert HTML email to plaintext for multipart emails. SMTP errors
are handled appropriately by retrying or falling through to the next email. There are
several configurable settings with defaults. DEFAULT_BULK_FROM_EMAIL specifies the
from address for email. EMAIL_HOST, EMAIL_PORT, EMAIL_HOST_USER, EMAIL_HOST_PASSWORD,
and EMAIL_USE_TLS are for the SMTP email backend settings.

![Resulting message](image/bulkemail-footer.png)

We use Amazon SES to send our mails.  We have SES configured to sign
messages on our behalf using DKIM to reduce the likelihood that they
will be treated by spam by the receiver.  Not the signature in the
following screenshot.

![Message signing](image/bulkemail-dkim.png)

## Optouts

Users have the option to opt out of course specific emails in the user dashboard through an Email Settings link for each course.

![Optout UI](image/bulkemail-optout.png)