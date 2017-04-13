## Web application checklist

Before you release a Rails application to production, you **must** go through this checklist.

Not all checklist items are necessary in every app.

If your not sure whether you should add some feature (e.g. Google Analytics) to the webapp your releasing, consult with the project manager and/or your team leader.

In your project management app (e.g. Productive, Trello...) create a task _Production Checklist_ and link to this handbook chapter. Finish it prior to production releasing.

## Checklist items:

* Are you uploading all files to S3 and not to the local (public/system) filesystem?
* Check that the app doesn't have any security issues on Code Climate. If it isn't on Code Climate, why isn't it? If there is a explicit reason why it isn't, check the security of your application with [Brakeman](https://github.com/presidentbeef/brakeman)
* Add bugsnag.
* Gzip the content with [Rack Deflater](http://robots.thoughtbot.com/content-compression-with-rack-deflater/)
* Check if indices are all setup - if you have the postgres `pg_stat_statements` extension enabled, check what are some of the slow queries and consult with the team how to decrease their execution time. Also checkout [lol_dba](https://github.com/plentz/lol_dba) - a static checker for indices based on foreign keys.
* Check if models have dependent actions on their associations

**HTML apps only:**

* Test the app on various browsers. The usual set of browsers is (this can vary from project to project): Chrome, FireFox, Opera, Safari, IE8+. Do this before the app goes to QA so they don't have to return the app with trivial mistakes.
* Go through a final design review with the designer.
* Handout the application to the QA team so they can try to find potential bugs before the client or customers do.
* Add the cookies_eu gem.
* Add google analytics.
* Add metatags.
* Add a favicon.
* Add a custom 404/500 page if necessary.
* Did you add credits (i.e.link to www.infinum.co and Developed by Infinum).

**Devops specific:**

* Is your application listed in Infinum's Catalog? Is all data up to date?
* Are we monitoring the app?
* Where's the DNS zone? If there are no dependencies (like Cpanel, lots of subdomains etc..), you're probably better off hosting it on our DNS servers (PointHQ) because it's faster to do changes then contacting a third party DNS hoster
* Lower TTL for all A/CNAME records to 300.
* Are you using SSL?
* Check if the log rotation is set up on the server.
