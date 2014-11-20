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

Right now, we only support CyberSource as the payment gateway. There is a an API in the shopping cart to allow for other payment providers, so this is an area of the system that can be extended. However, in the meantime, if you wish to use the Shopping Cart "out-of-the-box", users will have to acquire a merchant account with CyberSource.

To enable Paid Course Enrollments, the following configuration must be set in your environment:

In lms.env.json:

```
...
"FEATURES": {
...
"ENABLE_SHOPPING_CART": true,
"ENABLE_PAID_COURSE_REGISTRATION": true,
...
}
....
```

Also, in lms.auth.json (which contains production "secrets")

```
...
    "CC_PROCESSOR": {
        "CyberSource2": {
            "ACCESS_KEY": "{fill in with access key provided by CyberSource}", 
            "PROFILE_ID": "{fill in with profile id provided by CyberSource}", 
            "PURCHASE_ENDPOINT": "{fill in with endpoint url provided by CyberSource}", 
            "SECRET_KEY": "{fill in with secret key provided by CyberSource}"
        }
    }, 
    "CC_PROCESSOR_NAME": "CyberSource2",
... 
```

To configure a course to be a Paid Course Enrollment, a system administrator will need to go to the Django Admin website that is at the /admin URL. In order to gain access to this area of the Django website, the user must have 'superuser' rights as documented at https://github.com/edx/configuration/wiki/edX-Managing-the-Production-Stack.

Once you have logged into the Django Admin look for and click on the "Course modes" link. This will show all existing Course Mode that have been defined. At the top right of this page, there will be a button which says "Add course mode". Click that button.

Then you will see a form to fill out:

1) Course id: Please put the course id associated with this course. This will be in the format of "{org}/{course}/{run}", such as "foo/bar/baz". It must exactly match the course ID set up in Studio.

2) Mode slug: You must set this to "honor"

3) Mode display name: You can set this to however you wish to display this "mode". One recommended value would be "Honor Code"

4) Min Price: Fill in with the price of the course

5) Suggested Prices: Use the same price as with #4

6) Currency: this defaults to "usd" (US Dollars). You can change this to be a different ISO code (e.g. 'chf' for Swiss Francs)

7) Expiration Date/Date Time. It is recommended to leave these blank.

# Finance Admin Role Permissions

For Paid Course Registration courses, on the Instructor Dashboard a new tab 'eCommerce' will appear. For regular course staff accounts, this will include basic functionality to set up discounts (coupons). For additional capabilities, such as getting reports and to change the price of a course, a new "Course Access Role" has been defined.

Appropriate course staff members should be entered into a 'finance_admin' role for the course. To do so:

1) Go to the Django Admin site (/admin) as a superuser
2) Scroll down to find the "Course Access Role" (in the Student section). Click on the link.
3) Click "Add Course Access Role" button in the top right
4) Fill in the form. You can search for the user id (by email or username) using the search tool. Make sure the ORG and Course ID are filled in with the appropriate course ID and the ORG should be the same ORG that is part of the Course ID tuple. For the "Role" field, please set it to "finance_admin".

For the permission to take affect the user who was granted this role should log out and log back in.
