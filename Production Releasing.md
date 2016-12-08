# Web application checklist

Before you release a Rails application to production, you **must** go through this checklist.

Not all checklist items are necessary in every app.

If your not sure whether you should add some feature (e.g. Google Analytics) to the webapp your releasing, consult with the project manager and/or your team leader.

## Checklist items:

* Test the app on various browsers. The usual set of browsers is (this can vary from project to project): Chrome, FireFox, Opera, Safari, IE8+. HTML apps only.
* Go through a final design review with the designer. HTML apps only.
* Handout the application to the QA team so they can try to find potential bugs before the client or customers do. HTML apps only.
* Are you uploading all files to S3 and not to the local (public/system) filesystem?
* Did you add your application to the [Application list](https://github.com/infinum/internal-guides/blob/master/applications/Readme.md)
* Did you add monitoring via UptimeRobot?
* If the project is on code climate, check that it doesn't have any security issues. If the project isn't on code climate, check the security of your application with [Brakeman](https://github.com/presidentbeef/brakeman)
* Where's the DNS zone? If there are no dependencies (like Cpanel, lots of subdomains etc..), you're probably better off hosting it on our DNS servers (PointHQ) because it's faster to do changes then contacting a third party DNS hoster
* Lower TTL for all A/CNAME records to 300.
* Are you using SSL?
* Add the cookies_eu gem.
* Add bugsnag.
* Add google analytics.
* Add metatags.
* Add a favicon.
* Add a custom 404/500 page if necessary.
* Add deflater http://robots.thoughtbot.com/content-compression-with-rack-deflater/. HTML apps only.
* Did you add credits (i.e.link to www.infinum.co and Developed by Infinum). HTML apps only.
* Check if indices are all setup - checkout [lol_dba](https://github.com/plentz/lol_dba)
* Check if models have dependent actions on their associations
* Check if the log rotation is set up on the server.
