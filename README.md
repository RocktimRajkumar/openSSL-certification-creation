# OpenSSL Certification Authority (CA) on Ubuntu Server

OpenSSL is a free, open-source library that you can use for digital certificates. One of the things you can do is build your own CA (Certificate Authority).

A CA is an entity that signs digital certificates. An example of a well-known CA is `Verisign`. Many websites on the Internet use certificates for their HTTPS connections that were signed by `Verisign`.

Besides websites and HTTPS, there are some other applications/services that can use digital certificates. For example:

- VPNs: instead of using a pre-shared key you can use digital certificates for authentication.
- Wireless: WPA 2 enterprise uses digital certificates for client authentication and/or server authentication using PEAP or EAP-TLS.

Instead of paying companies like Verisign for all your digital certificates. It can be useful to build your own CA for some of your applications. In this lesson, you will learn how to create your own CA.

## Prerequisites
 - Ubuntu OS
 - Before we configure OpenSSL, I like to configure the hostname/FQDN correctly and make sure that our time, date and timezone is correct.

    Let’s take a look at the hostname:

    ```
    $ hostname
    rock
    ```

    My hostname is `rock`. Let’s check the FQDN:

    ```
    $ hostname -f
    rock
    ```

    It’s also `rock`. Let’s change the FQDN; you need to edit the following file for this:
    ```
    $ sudo vim /etc/hosts 
    ```
    Change the following line:
    ```
    127.0.1.1       rock
    ```
    To:
    
    ```
    127.0.1.1       rock.mywebsite.local rock
    ```
    Let’s verify the hostname and FQDN again:
    
    ```
    $ hostname
    rock
    ```
    ```
    $ hostname -f
    rock.mywebsite.local
    ```
    Our hostname and FQDN is now looking good.


## Root CA

### Create the folder structure
The `/root/ca` folder is where we will store our private keys and certificates.

```
$ mkdir /root/ca
$ cd /root/ca

$ mkdir newcerts
$ mkdir certs
$ mkdir crl
$ mkdir private
$ mkdir requests
$ touch index.txt
```
`certs` -> Directory contains all the certificate

`requests` -> Directory contains all the requests

`index.txt` -> This is where OpenSSL keeps track of all signed certificates:

`private` -> This is where all the private key store

**The first thing we have to do is to create a root CA. This consists of a private key and root certificate. These two items are the `identity` of our CA.**

## File to store serial number of the certificate
Each signed certificate will have a serial number.  I will start with number `1234`:

```
$ echo '1234' > serial
```

### Create a root private key

```
$ openssl genrsa -aes256 -out private/cakey.pem 4096
```
The root private key that I generated is 4096 bit and uses AES 256 bit encryption. It is stored in the private folder using the `cakey.pem` filename.

Provide the pass phrase.

### Root Certificate creation
We can now use the root private key to create the root certificate:

```
$ openssl req -new -x509 -key /root/ca/private/cakey.pem -out cacert.pem -days 3650
```
**`Note :`** You have to enter the same pass phrase of `cakey.pem` while creating the private key.



### Security

Protecting your CA is important. Anyone that has access to the private key of the CA will be able to create trusted certificates.

One of the things you should do is reducing the permissions on the entire /root/ca folder so that only our root user can access it:
```
$ chmod 600 -R /root/ca
```

### Change configuration of openssl.cnf file
```
$ sudo vim /usr/lib/ssl/openssl.cnf
```
Change dir path from ./demoCA to /root/ca

```
####################################################################
[ CA_default ]

dir             = /root/ca              # Where everything is kept
certs           = $dir/certs            # Where the issued certs are kept
crl_dir         = $dir/crl              # Where the issued crl are kept
database        = $dir/index.txt        # database index file.
```

Some fields like country, state/province, and organization have to match. If you are building your CA for a lab environment like I am then you might want to change some of these values:

```
policy          = policy_match

# For the CA policy
[ policy_match ]
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional
```

Right now we have our working Certificate Authority. And it's ready to start signing out the certificates.


## Create a certificate

Our root CA is now up and running. Normally when you want to install a certificate on a device (a web server for example), then the device will generate a CSR (Certificate Signing Request). This CSR is created by using the private key of the device.

On our CA, we can then sign the CSR and create a digital certificate for the device.

Another option is that we can do everything on our CA. We can generate a private key, CSR and then sign the certificate…everything “on behalf” of the device.

That’s what I am going to do in this example; it’s a good way to test if your CA is working as expected.

I’ll generate a private key, CSR and certificate for an imaginary `web server`.


### Generating a private key for the webserver.
Let’s use the requests folder for this:
```
$ cd requests
$ openssl genrsa -aes256 -out webserver.pem 2048
```

### Creating a CSR
With the private key, we can create a CSR
```
$ openssl req -new -key webserver.pem -out webserver.csr
```

### Signing the CSR
```
$ openssl ca -in webserver.csr -out webserver.crt
```

### Moving the file the correct folder
```
$ mv webserver.crt /root/ca/certs/
$ mv webserver.pem /root/ca/private/
```

## Verification
Here’s the index.txt file:

```
# cat /root/ca/index.txt
V       170401090859Z           1234    unknown /C=NL/ST=North-Brabant/O=Networklessons/CN=some_server.networklessons.local/emailAddress=admin@networklessons.local
```
Above you can see the certificate that we created for our web server. It also shows the serial number that I stored in the serial file. The next certificate that we sign will get another number:

```
# cat /root/ca/serial
1235
```

**`Note :`** Windows doesn’t recognize the **.PEM** file extension so you might want to rename your certificates to **.CRT**.

Rename the `cacert.pem` to `cacert.crt` and run it on windows machine.

Initally it will show not trusted, once installed installed it will be fix.

While installing please select certification store as `Trusted Root Certification Authorities`.

Windows will give you one more big security warning, click Yes to continue.

The root certificate is now installed and trusted. Now open the certificate that we assigned to `webserver`, i.e `webserver.crt`

You can see that it was issued by our root CA, it’s valid for one year. When you look at the certification path then you can see that Windows trusts the certificate.

