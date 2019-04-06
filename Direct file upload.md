# General

Direct file upload is the process of uploading files to a cloud storage service.

Mobile or web frontends often upload files through the app server, which means that the file does a double hop: from the frontend to the backend, then from the backend to the cloud storage service.

Direct upload solves this double-hop performance problem by giving one-time credentials to the frontend app to upload files directly to the cloud, and it sends out references to those files to the backend.

This is highly useful if you want to speed up the uploading process and improve user experience.

# The workflow

We use Amazon's S3 storage service. [This documentation](https://www.ironin.it/blog/store-your-files-on-s3-using-the-ruby-shrine-gem-part-2.html)
covers all the code and configuration files necessary to support direct upload to S3.
