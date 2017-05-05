# Password managment

Infinum uses [LastPass] for password managment. It is a cloud based password managment system. You will use it for storing passwords for Amazon, Mailgun, Databases and a lot of other services where more then one person needs to login, as well as for accounts that you only use.

## Account creation

First thing you need to know is how to get your hands on an account. Unfortunately LastPass doesn't have automated account creation, so you will need to ping [Josip Bišćan] for your invite.

Once you get invited, first thing you have to do is **enable 2-step** verification for your account. That is done by logging in to the web, selecting account options in your vault, and then under Multifactor Options, you can select your prefered authentication option.

Depending on your position, you should recieve access to shared folders. The permissions for certain levels of accounts is as follows:
    
 - Management - Administrators
 - Administrators (DevOps & Team leads) - Can Administer
 - Developers (everyone else) - Can Read/Write (Shared Websites, Shared Mailgun etc)

Basic principle, if you can't login, you are not supposed to be there. If you think you are, please contact your Team lead.

## Proper usage

Every account on any service you create that is bussiness related should go on LastPass. The proper way to create an account is to use a generated password. You can use your email, or if you think the account is a really generic one, you can request a new email to register the account to that mail instead. 

**Passwords should be at least 12 characters long, and you should generate them with every option for character sets included**

After you create the account, please save the information to one of the shared folders to which it belongs.

 - Every device and gadet from the office goes to Shared-OfficeNetwork
 - Every password for root access to databases goes to Shared-Databases
 - Every Mailgun account goes to Shared-Mailgun
 - Every website goes to the Shared-Websites folder (depending on if it's an admin account, it can go to the admin folder instead)


## Tutorial for IAM

First step is to save a site to LastPass

![step1](/public/img/password_managment_step1.png "step1")

The form should look like this

![step2](/public/img/password_managment_step2.png "step2")

Next step is to trim the unnecessary fields, such as the MFA token

![step3](/public/img/password_managment_step3.png "step3")

And that's that, you should have everything setup


[LastPass]:https://lastpass.com/
[Josip Bišćan]:mailto://josip.biscan@infinum.hr
