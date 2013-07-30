# Bulk Email

Course staff can now compose emails and send them to course staff.  The
key features are:

* Ability to compose HTML messages in the site.
 
* MIME multipart mails are sent
 
* Ability to efficiently to send to very large classes by fanning out
  jobs to worker machines over a job queue (celery).

* Mails are CAN-SPAM compliant: messages automatically have an optout link in the footer.

Credit goes to [Jason Bau](jb) for developing the
feature in Class2Go.  Students [Kevin Lau](kl) and [Akshay
Jaggadish](aj) took that code and ported to OpenEdX over Summer
2013.

  [jb]: mailto:jbau@stanford.edu
  [kl]: mailto:kevluo@stanford.edu
  [aj]: mailto:akshayj@berkeley.edu

![Creating an email](image/bulkemail-editor.png)

![Resulting message](image/bulkemail-footer.png)

We use Amazon SES to send our mails.  We have SES configured to sign
messages on our behalf using DKIM to reduce the likelihood that they
will be treated by spam by the receiver.  Not the signature in the
following screenshot.

![Message is signed](image/bulkemail-dkim.png)
