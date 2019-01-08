## Fiscalizer

We use the [fiscalizer gem](https://github.com/infinum/fiscalizer) to transfer various document types from a business entity to the tax authorities.

### Setup & Usage

Everything about setup and usage of this gem is explained at [fiscalizer gem](https://github.com/infinum/fiscalizer).

### Certificates

#### Environments:

+ ##### Production environment

Requires only one certificate - application certificate. It is most often called `FISCAL_1.p12`. It contains the public and private key, and the CA (`Certificate authority`) certificate.

Secrets file example:

```Ruby
fiscalization_app_cert_path: 'cert/production/FISKAL_1.p12'
fiscalization_ca_cert_path: ''
```


+ ##### Development & staging environment

Requires 2 certificates - both application and server certificate. The application certificate (`FISCAL_1.p12`) contains the public and private key. The server certificate (most often `fina_ca.pem`) contains the required CA certificates.

Secrets file example:

```Ruby
fiscalization_app_cert_path: 'cert/development/FISKAL_1.p12'
fiscalization_ca_cert_path: 'cert/development/fina_ca.pem'
```

#### Application certificate unpacking

To view the content of the application certificate locally, you need to unpack it with this command:

```
openssl pkcs12 -in FISKAL_1.p12 -out FISKAL_1.pem -nodes
```

After running this command, you are prompted to input the `fiscalization password`.

#### Create your own certificate

If you need to test some features that require certificates but you don't have them and your request cannot be sent, you can create a certificate following this [tutorial](https://datacenteroverlords.com/2012/03/01/creating-your-own-ssl-certificate-authority/).
Once done, create a `.p12` file following these two steps:

1. Copy the private key and SSL certificate to a plain text file. The private key should be at the top with the SSL certificate below. In the example, we use "filename.txt" as the name of the file containing the private key and SSL certificate.

2. `openssl pkcs12 -export -in filename.txt -out filename.p12`
