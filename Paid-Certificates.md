## Overview

This feature enables the ability to set different course tracks and process payments for them.

## Django configuration settings

* **ENABLE_PAID_COURSE_REGISTRATION** - Boolean toggle to enable flow for payments for course registration
* **ENABLE_SHOPPING_CART** - Boolean toggle to enable the shopping cart page
* **CC_PROCESSOR** - Dict of dict with Credit Card Processor information. Currently support for CyberSource is included in the platform. See lms/djangoapps/shoppingcart/processors.
* **MULTIPLE_ENROLLMENT_ROLES** - Boolean toggle to allow users to enroll with methods other than just honor code certificates
* **PAID_COURSE_REGISTRATION_CURRENCY** - Currency for paid courses. E.g. ['usd', '$']
* **STORE_BILLING_INFO** - Boolean toggle for storing detailed billing information
* **CC_MERCHANT_NAME** - company name that is displayed on the order confirmation email 
* **PAYMENT_SUPPORT_EMAIL** - Email address for refund requests, etc.
* **PAYMENT_REPORT_GENERATOR_GROUP** - Members of this group are allowed to generate payment reports

## Other info

* Enabling Course Modes
 * Get the course_id for the course you want to add modes to (ex. MITx/6.002x/2013_Spring)
 * Visit the studio admin dashboard with a user that has superuser access. (There's a column in the auth_user database called is_superuser)
 * Click 'Course Modes'
 * Add a new course mode info
  * Mode slug is the name of the mode (e.g. 'verified', 'audit', 'honor')
  * The min price is an integer field. 
  * The suggested prices fields takes a comma separated value: ex. "25,50,100" - set the expiration_date to the date when you no longer want to allow people to sign up for this course mode (all dates are in UTC)
 * Create other modes, e.g. mode slugs of 'audit' and 'honor' with a min price of 0 and no suggested prices.
 * When you try to register for the course, you should now be given an option of which mode to choose.

* Reports - See lms/djangoapps/shoppingcart/reports.py. Currently available reports:
 * Refunds
 * Itemized Purchases
 * University Revenue Share
 * Certificate Status
