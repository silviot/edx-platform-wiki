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

```
num_course_enrollments = number_of_purchased_paidcourseregistrations + 
  number_of_redeemed_registration_codes + number_of_course_staff + 
  number_of_manually_enrolled_students
```

Ideally number_of_manually_enrolled_students = 0 as that feature typically would not be used in a shopping cart enabled course.

So, in order to perform the audit via SQL queries, here are the relevant elements. Rather than combining it into one mega-query, each element will have a separate query. Please substitute the correct course_id for your query.

```
select count(*) as total_enrollments from student_courseenrollment 
where course_id='{course_id}';

select count(*) as number_of_course_staff from student_courseenrollment ce 
where course_id='{course_id}' and ce.user_id in 
(select distinct(user_id) from student_courseaccessrole where 
(role='instructor' or role='staff') and course_id='{course_id}');

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

```

Note that in the last two queries, we're filtering out any course staff which might have also purchased (or redeemed a registration code) to prevent "double_counting". It is not uncommon for course staff to "test" their course to make sure things are working properly, therefore it is prudent to filter them out because they are already being counted in the number of course staff query.

Unfortunately, there is no way to audit the count of users that were manually enrolled in a course. As mentioned earlier, ideally this number should be 0 as students should not be manually enrolled in a shopping cart enabled course.

IMPORTANT: If you run these queries on a course which is has open enrollments, please remember that students could be registering for courses *at the same time* you are doing the audit. So there could be skews in the numbers depending on how active course enrollments are. It is recommended to work off of a database snapshot or to do auditing when the enrollment period is closed.

## Payment Processor audits

Likewise, a typical audit would like to verify that the financial totals in the Shopping Cart database matches the totals that the payment processor service (e.g. CyberSource) has. This is one of the challenges of having a decoupled system: one database for the Open edX app and a separate database for the payment provider. Ideally, the totals should match between the databases.

One current limitation with the Open edX Shopping Cart is that refunds/chargebacks are not recorded in the Open edX Shopping Cart database, but it *is* recorded in the Payment Gateway. Therefore, refunds/chargebacks become one area of reconciliation. Please note that we hope to have a deeper integration of Open edX reporting capabilities to pull refund information from CyberSource, but it does not yet currently exist.

So, in short, the business logic is simply:

```
total_in_payment_gateway_including_refunds = total_paidcourseregistrationitem + 
  total_regcodeitem - refunds_unrecorded_in_shoppingcart_db
```
refunds_unrecorded_in_shoppingcart_db is - unfortunately - not queryable in the Shopping Cart database. So it is preferable to start with something more tangible:

```
total_in_payment_gateway_including_refunds + refunds_in_payment_gateway = 
  total_paidcourseregistrationitem + total_regcodeitem
```

To query total_paidcourseregistrationitem and total_regcodeitem, do:

```
select sum(oi.qty*oi.unit_cost) as total_paidcourseregistrationitem from 
shoppingcart_paidcourseregistration pcr join shoppingcart_orderitem oi on 
pcr.orderitem_ptr_id=oi.id where oi.status='purchased' 
and pcr.course_id='{course_id}';

select sum(oi.qty*oi.unit_cost) as total_regcodeitem from 
shoppingcart_courseregcodeitem cri join shoppingcart_orderitem oi 
on cri.orderitem_ptr_id=oi.id where oi.status='purchased' 
and cri.course_id='{course_id}';
```

Adding total_paidcourseregistrationitem + total_regcodeitem *should* equal whatever reporting your payment processor (CyberSource) if refunds/chargebacks are not included in the payment processor report. If refunds/chargebacks are included in the payment processor, then it is recommended that you compute the amount of refunds and add them back in.

So if your payment processor generated report looks something like:

```
transaction_num: 1   total: $100
transaction_num: 2   total: $100
transaction_num: 3   total: $200
transaction_num: 4   total: $-100 (refund)
```

The total would be $300. However, Shopping Cart queries would come to $400, since it is not aware of the refunded purchase. Therefore, you should add back in the refunded $100, and then the totals will line up.

IMPORTANT: If you run these queries on a course which is has open enrollments, please remember that students could  be registering for courses *at the same time* you are doing the audit. So there could be skews in the numbers depending on how active course enrollments are. It is recommended to work off of a database snapshot or to do auditing when the enrollment period is closed. Also, there might be processing latency with the 3rd party payment processor (like CyberSource) where the change is immediate in the Shopping Cart, but the report in the payment processor report may lag behind (minutes? hours? Hard to tell unfortunately).

## Computing total number of seats sold

It is often a useful metric to compute the total number of seats sold. In terms of business logic, this is:

```
num_seats_sold = total_purchased_paidcourseregistrations +
 num_of_regcodes_generated
```

The num_of_regcodes_generated is a combination of multi-seat purchases (RegCodeItems) as well as registration codes generated via Invoices. Note that registration codes generated via Invoices can be set to any arbitrary price, so keep that in mind when reconciling financials. Also Invoices are settled "out-of-band" and therefore it cannot be reconciled against the payment gateway.

In terms of SQL, use:

```
select count(*) as total_purchased_paidcourseregistrations from 
shoppingcart_paidcourseregistration pci join shoppingcart_orderitem oi 
on oi.id=pci.orderitem_ptr_id where oi.status='purchased' and 
pci.course_id='{course_id}'

select count(*) as num_of_regcodes_generated from 
shoppingcart_courseregistrationcode where course_id='{course_id}'
```

Please note that if course staff perform "test purchases" or "test registration codes", those staff people will be counted in these totals (and that is why we filter them out in the Enrollment audit queries up above). 