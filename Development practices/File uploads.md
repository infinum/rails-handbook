Our standard file upload setup includes:

* using Shrine gem
* direct S3 uploads
* [S3](https://aws.amazon.com/s3/) as file storage for production and staging servers, and disk storage for development.

## Shrine
There are several file upload gems out there. We use [Shrine](https://github.com/janko-m/shrine) because of its flexibility and maintainability.
It is a general-purpose file uploader gem. Since it has no direct dependency,
__it can be used in both Rails and non-Rails projects.__

Official docs can be found [here](https://shrinerb.com/)
There is also [a demo application](https://github.com/erikdahlstrand/shrine-rails-example).

### Plugins
Shrine features a rich plugin system which makes it incredibly easy to change its behavior to suit your needs.

Here are some of the plugins we use often:

* [activerecord](https://shrinerb.com/docs/plugins/activerecord)
* [bakgrounding](https://shrinerb.com/docs/plugins/backgrounding)
* [cached\_attachment\_data](https://shrinerb.com/docs/plugins/cached_attachment_data) - for forms
* [derivatives](https://shrinerb.com/docs/plugins/derivatives)
* [determine\_mime\_type](https://shrinerb.com/docs/plugins/determine_mime_type)
* [presign_endpoint](https://shrinerb.com/docs/plugins/presign_endpoint) - for direct S3 uploads
* [upload_options](https://shrinerb.com/docs/plugins/upload_options)
* [url_options](https://shrinerb.com/docs/plugins/url_options)
* [validation](https://shrinerb.com/docs/validation)
* [validation_helpers](https://shrinerb.com/docs/plugins/validation_helpers)

### Security
Files stored on S3 are private by default. This means file URLs will be signed and they will expire after some specified time.

It's best to explicitly set the expiration time for **each uploader class** using [url_options](https://shrinerb.com/docs/plugins/url_options) plugin.

Some files need to be public, i.e. albums' covers. In that case, set the `acl` to `public-read` via [upload_options](https://shrinerb.com/docs/plugins/upload_options) plugin for that uploader class.

### Other guidelines

- Use **jsonb** data type for file columns when possible.

- If you're uploading images, you should process them (file compression) and create **multiple versions** (derivatives in Shrine) of different sizes. Consult with frontend devs on required dimensions.
**Backgrounding** plugin should be combined with the processing for a better user experience.

- Always validate **mime type** for uploaded files, and extension if needed.

- Shrine doesn't automatically delete files from cache storage when moving them to store storage. Tell a DevOps to **set a lifecycle policy** with an appropriate amount of time **for cache storage** prefix.

## Direct S3 upload
Mobile or web front ends often upload files through the app server, which means that the file does a double hop: from the frontend to the backend, then from the backend to the cloud storage service.

Direct upload solves this double-hop performance problem by giving one-time credentials to the frontend app to upload files directly to the cloud, and it sends out references to those files to the backend.

This is extremely useful if you want to speed up the uploading process and improve user experience.

To implement direct S3 upload with Shrine, read the instructions [here](https://shrinerb.com/docs/direct-s3).

## Accessing files on S3
To access files on s3, ask someone from the DevOps team for the S3 console access.
