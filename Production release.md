## Web application checklist

Before you release a Rails application to production, you **must** go through this checklist.

Not all checklist items are necessary in every app.

If you're not sure whether you should add some feature (e.g., Google Analytics) to the web app you're releasing, consult with the project manager and/or your team leader.

In your project management app (e.g., Productive, Trello...), create a _Production Checklist_ task and link to this handbook chapter. Finish it prior to production release.

## Checklist items:

* Are you uploading all files to S3 and not to the local (public/system) filesystem?
* Check that the app doesn't have any security issues on Code Climate. If it isn't on Code Climate, why isn't it? If there is a specific reason for not using Code Climate, check the security of your application with [Brakeman](https://github.com/presidentbeef/brakeman).
* Check that Bugsnag is included.
* Gzip the content with [Rack Deflater](http://robots.thoughtbot.com/content-compression-with-rack-deflater/).
* Check that indices are all set up—if you have the Postgres `pg_stat_statements` extension enabled, check what are some of the slow queries and consult with the team on ways to decrease their execution time. Also, check out [lol_dba](https://github.com/plentz/lol_dba)—a static checker for indices based on foreign keys.
* Check if models have actions dependent on their associations.

**HTML apps only:**

* Test the app on various browsers. The usual set of browsers is (this can vary from project to project): Chrome, FireFox, Opera, Safari, and IE8+. Do this before the app goes to QA so they don't have to return the app because of trivial mistakes.
* Go through a final design review with the designer.
* Pass on the application to the QA team so they can try to find potential bugs before the client or customers do.
* Add the cookies_eu gem.
* Add Google Analytics.
* Add metatags.
* Add a favicon.
* Add a custom 404/500 page if necessary.
* Did you add credits (i.e., a link to www.infinum.co and Developed by Infinum).

**Devops-specific:**

* Is your application listed in Infinum's Catalog? Is all data up-to-date?
* Are we monitoring the app?
* Where's the DNS zone? If there are no dependencies (such as Cpanel, lots of subdomains, etc.), you're probably better off hosting it on our DNS servers (PointHQ), because it's faster to make changes than contact a third-party DNS host.
* Lower TTL for all A/CNAME records to 300.
* Are you using SSL?
* Check if the log rotation is set up on the server.
