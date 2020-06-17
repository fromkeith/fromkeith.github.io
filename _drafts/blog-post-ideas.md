# Development Mistakes I've made (draft)
_(and things I wished existed earlier)_

* dynamodb too early
  * scaling. huge delay
  * paying for unused capacity
  * ondemand did not exist
* golang - chromium thing
  * electron not around or at least not popular yet
  * updating it...
* using a nat to download from s3
* existed earlier... docker as "server version control"
  * having to copy files onto an ami. save ami.
* typescript
  * i wished it existed earlier
  * rewriting an uploading module
* native android vs js
  * friend to develop in native
  * maintaince was harder
* photo resizing
  * lambda to the rescue
  * early lambda issue of re-use in badly configured instance
* autoscaling off of sqs
  * number of message visible can be delayed
* using ec2 role permission to sign s3 uploads for 3rd party
* not using a cms
  * having to update when cofounder wanted to change homepage
* using a wordpress hoster that charges for usage volume
  * get attacked by bot. tough luck



# Things that annoy me

* Google Cloud's permission model
  * having a JSON or pk12 file is a PITA
  * its not obvious what roles appengine uses or how to limit them
  * makes the upfront cost to test things out feel huge
  * are projects how I separate permissions?
* Azure
  * where the fuck is the login page. how do i get back to where my devops are. bookmars
* IBM
  * credit given. awesome. but nothing to use it on.
* Square
  * docs were inconsistent and lacking information
  * don't provide a way for developers to get a cut of instore
  * no way to reliably validate an instore transaction. delays, lack of ids, etc
* DialogFlow
  * api documentation. wtf? i just want to test an intent. just to understand the url i need to research stuff
* ECS Run Task
  * stop creating new seurity groups.
  * run more like doesn't populate anything
* AWS pricing
  * hidden costs on pricing pages.
  * its X dollars to run this long. oh and the cloudwatch pricing is XX
* AWS Support
  * we recommend using IAM roles
  * we cannot talk to users of IAM roles only the root account
* AWS permissions (policies vs inline)
  * property of least priveledge
  * policies are against that
  * inline is heavily discourage

# preferences

* gulp vs runfile
  * watching slows down dev speed significantly on large projects
