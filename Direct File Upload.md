# General

Direct file upload is the process of uploading files to a cloud storage service.

Often times mobile or web frontends upload files through the app server which means that the file does a double hop: from the frontend to the backend, then from the backend to the cloud storage service.

Direct upload resolves this double hop performance problem by handing out one-time credentials to the frontend app to upload files directly to the cloud and sends out references to those files to the backend.

It is highly useful if you want to speed up the uploading process and to improve user experience.

# The Workflow

We use Amazon's S3 storage service. [This documentation](https://www.ironin.it/blog/store-your-files-on-s3-using-the-ruby-shrine-gem-part-2.html)
covers all the code and configuration files needed for supporting direct upload to S3.
