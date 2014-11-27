# Background

The Open edX platform supports a number of use cases to enable eCommerce. The most typical use cases are:

- As an administrator of a run of a course, I would like to only allow students from enrolling in a course unless they purchase the course with a Credit Card. 
- As an administrator of a run of a course, I would like to set up discounting via coupons.
- As an administrator of a run of a course, I would also like to get some financial reporting on this course.
- As a user, I wish to enter myself into a course that has an associated price (single-seat purchase).
- As a user, I wish to enter a group of people into a course that has an associated price (multiple-seat purchase).

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

2) Mode slug: You must set this to "honor".

3) Mode display name: You can set this to however you wish to display this "mode". One recommended value would be "Honor Code".

4) Min Price: Fill in with the price of the course.

5) Suggested Prices: Use the same price as with #4.

6) Currency: this defaults to "usd" (US Dollars). You can change this to be a different ISO code (e.g. 'chf' for Swiss Francs).

7) Expiration Date/Date Time. It is recommended to leave these blank.

# Multi-seat purchases (CourseRegCodeItem)

It is possible for the Open edX platform to allow for a user to purchase multiple "Registration Codes" to a course. In the single seat purchase, PaidCourseEnrollment as described above, the user who completed the transaction is immediately enrolled in the course.

However, when multiple seats are purchased, the purchaser is *not* automatically enrolled in the course. Instead, the platform will generate short random Registration Codes, which can be redeemed by a different user (or the purchaser him/herself).

When a purchaser completes a multi-seat transaction, he/she will receive an email with a list of these Registration Codes. Furthermore, these Registration Codes will be listed on the shoppingcart Receipt page after the payment processing occurs.

# Invoicing

It is also possible for a course administrator to generate the above Registration Codes and associate it with an Invoice. An Invoice is an "out-of-band" means to purchase multiple seats to a course. The distinction is that with multi-seat purchases (CourseRegCodeItem) the transaction is completed with a Credit Card. For Invoices, the transaction is completed through other means.

Invoices - for now - can only be generated by a course administrator who has the 'finance_admin' role (see below). This is done through the Instructor dashboard, on the e-commerce tab. The admin must fill in a form which includes a lot of information about who is purchasing the seats, as well as how many seats were sold at what price.

When the Invoice form is filled out, the platform will generate as many Registration Codes as was requested and send them to the appropriate people via email. Also a .CSV file will be generated and downloaded to the admin's browser.

# Finance Admin Role Permissions

For Paid Course Registration courses, on the Instructor Dashboard a new tab 'eCommerce' will appear. For regular course staff accounts, this will include basic functionality to set up discounts (coupons). For additional capabilities, such as getting reports and to change the price of a course, a new "Course Access Role" has been defined.

Appropriate course staff members should be entered into a 'finance_admin' role for the course. To do so:

1) Go to the Django Admin site (/admin) as a superuser.

2) Scroll down to find the "Course Access Role" (in the Student section). Click on the link.

3) Click "Add Course Access Role" button in the top right.

4) Fill in the form. You can search for the user id (by email or username) using the search tool. Make sure the ORG and Course ID are filled in with the appropriate course ID and the ORG should be the same ORG that is part of the Course ID tuple. For the "Role" field, please set it to "finance_admin".

For the permission to take affect the user who was granted this role should log out and log back in.

# Auditing

If you use the shopping cart features on a course, you may wish to perform some audits from time to time. Two areas to audit are:

- Are all *student* course enrollments accounted for via transactions (note, staff enrollments - which are free - should not count)?

- Are the eCommerce totals in the shopping cart database match what is in the payment processor?

## Course Enrollment audits

In essence, the number of students (not course staff) enrolled in a course should equal the number of seats purchased (and redeemed, if applicable).

The business logic would have this equation:

num_course_enrollments = number_of_purchased_paidcourseregistrations + number_of_redeemed_registration_codes + number_of_course_staff + number_of_manually_enrolled_students

Ideally number_of_manually_enrolled_students = 0 as that feature typically would not be used in a shopping cart enabled course.

So, in order to perform the audit via SQL queries, here are the relevant elements. Rather than combining it into one mega-query, each element will have a separate query. Please substitute the correct course_id for your query.

```
select count(*) as total_enrollments from student_courseenrollment 
where course_id='{course_id}';

select count(*) as total_purchased_paidcourseregistrations from 
shoppingcart_paidcourseregistration pci join shoppingcart_orderitem oi 
on oi.id=pci.orderitem_ptr_id where oi.status='purchased' and 
pci.course_id='{course_id}' and oi.user_id not in 
(select distinct(user_id) from student_courseaccessrole where 
(role='instructor' or role='staff') and course_id='{course_id}');

select count(*) as number_of_redeemed_registration_codes from 
shoppingcart_registrationcoderedemption rcr join 
shoppingcart_courseregistrationcode crc on rcr.registration_code_id=crc.id 
where crc.course_id='{course_id}' and rcr.redeemed_by_id not in 
(select distinct(user_id) from student_courseaccessrole where 
(role='instructor' or role='staff') and course_id='{course_id}');

select count(*) as number_of_course_staff from student_courseenrollment ce 
where course_id='{course_id}' and ce.user_id in 
(select distinct(user_id) from student_courseaccessrole where 
(role='instructor' or role='staff') and course_id='{course_id}');
```

Unfortunately, there is no way to audit the count of users that were manually enrolled in a course.

Note that this above set of queries can cause some limited "double counting" if:

- A course staff also does a purchase (say for testing purposes)
- A course staff redeems a RegistrationCode