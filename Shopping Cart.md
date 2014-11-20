# Background

The Open edX platform supports a number of use cases to enable eCommerce. The most typical use cases are:

- As an administrator of a run of a course, I would like to only allow students from enrolling in a course unless they purchase the course with a Credit Card. 
- As an administrator of a run of a course, I would like to set up discounting via coupons
- As an administrator of a run of a course, I would also like to get some financial reporting on this course.
- As a user, I wish to enter myself into a course that has an associated price (single-seat purchase)
- As a user, I wish to enter a group of people into a course that has an associated price (multiple-seat purchase)

Note that edX.org has additional use-cases regarding the purchasing of "Verified Certificates", which will not be covered in this Documentation, as the UX flows are different.

This documentation will be for users of this feature as it currently exists and not developer documentation who may wish to extend the implementation.

# Acknowledgements

This work was originally contributed to Open edX from the group at Stanford University. There have been many additions by edX employees and Open Source contributors since the original submission.

# Paid Course Enrollments

When a course administrator wishes to only allow enrollments in a course after the user completes a financial transaction through a Credit Card processor, that is called a PaidCourseEnrollment.

To configure a course to be a Paid Course Enrollment, a system administrator will need to go to the Django Admin website that is at the /admin URL. In order to gain access to this area of the Django website, the user must have 'superadmin' rights as documented at https://github.com/edx/configuration/wiki/edX-Managing-the-Production-Stack.

Once you have logged into the Django Admin site you will see a screen which looks something like:


