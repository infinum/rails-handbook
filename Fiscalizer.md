## Fiscalizer

We are using the [fiscalizer gem](https://github.com/infinum/fiscalizer) to transfer various document types from a business entity to the tax authorities.

### Setup & Usage

Everything about setup and usage of this gem is explained on [fiscalizer gem](https://github.com/infinum/fiscalizer).

### Certificates

#### Environments:

+ #### Production environment

Requires only one certificate - application certificate. It is most often called `FISCAL_1.p12`. It contains the public and private key, and the CA (`Certificate authority`) certificate.

Secrets file example:

```Ruby
fiscalization_app_cert_path: 'cert/production/FISKAL_1.p12'
fiscalization_ca_cert_path: ''
```


+ #### Development & Staging environment

Requires 2 certificates - both application and server certificate. The application certificate (`FISCAL_1.p12`) contains the public and private key. The server certificate (most often `fina_ca.pem`) contains the required CA certificates.

Secrets file example:

```Ruby
fiscalization_app_cert_path: 'cert/development/FISKAL_1.p12'
fiscalization_ca_cert_path: 'cert/development/fina_ca.pem'
```

+ #### Application Certificate unpacking

To view the content of the application certificate locally, you need to unpack it with this command:

```
openssl pkcs12 -in FISKAL_1.p12 -out FISKAL_1.pem -nodes
```

After this command, you are prompted to input the `fiscalization password`.

+ #### Updating from 0.x to 1.x

1. `bundle update fiscalizer`
2. Add application and server certificate (`FISKAL_1.p12` and `fina_ca.pem`) for development
3. Change fiscalization option in your fiscalization service
From

```Ruby
{
  certificate_p12_path: secrets['certificate_p12_path'],
  certificate_path: secrets['certificate_path'],
  url: secrets['url'],
  certificate_issued_by: secrets['certificate_issued_by'],
  password: secrets['password']
 }
```

To

```Ruby
{
  app_cert_path: secrets['app_cert_path'],
  ca_cert_path: secrets['ca_cert_path'],
  password: secrets['password'],
  demo: !Rails.env.production?
 }
```

4. Change secrets file

From

```Ruby
  certificate_p12_path: ...
```

To

```Ruby
app_cert_path: ...
ca_cert_path: ...
```

Check an example [here](https://bitbucket.org/infinum_hr/web_tvornica_snova/pull-requests/327/feature-new-fiscalizer/diff)

### Create own certificate

If you need to test some features that require certificates but you don't have them and your request won't be send, you can create a certificate following this [tutorial](https://datacenteroverlords.com/2012/03/01/creating-your-own-ssl-certificate-authority/).
Once done, create a `.p12` file following this two steps:

1. Copy the private key and SSL certificate to a plain text file. The private key should go on top with the SSL certificate below. In the example, we use "filename.txt" as the name of the file containing the private key and SSL certificate.

2. `openssl pkcs12 -export -in filename.txt -out filename.p12`
