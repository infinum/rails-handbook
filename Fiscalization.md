## Issuing a new DEMO certifcate

* Person responsible for Infinum certificates (this is Luka Višić at the time of writing this article) has to go to FINA to request a new DEMO certificate. [Details](http://www.fina.hr/Default.aspx?sec=1542)
* FINA will send a referent key and PIN number
* Find a Windows machine with IE
* Install latest Java
* Install FINA certificates (I at long last found them here: http://rdc.fina.hr/download/fina-install.zip)
* Go to https://demo-mojcert.fina.hr/finacms/
* Click on 'Registriraj Se'
* Choose tab 'Preuzimanje Soft certifikata' and enter the referent nubmer and PIN number (Autorizacijski kod)
* Once logged in, just select new certificate and click on 'izdavanje', where you will be prompted to choose a password
* After choosing a password the Certificate will begin downloading

## Current DEMO certificate

* Current certificate and password are located on GDrive/Infinum Everyone/Rails Team/Fiskalizacijski DEMO cert

## Using DEMO certificate with fiscalizer gem

Be sure to use `> 1.0.0` version of [fiscalizer gem](https://github.com/infinum/fiscalizer/).
You need to set the following secrets:

```
fiscalization_certificate_p12_path: cert/development/FISKAL_1.p12
fiscalization_certificate_path: cert/development/fina_ca.pem
fiscalization_password: neki_pass
fiscalization_url: https://cistest.apis-it.hr:8449/FiskalizacijaServiceTest
fiscalization_certificate_issued_by: 'OU=DEMO,O=FINA,C=HR'
```
