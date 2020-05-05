# Create OpenSSL certificate in ubuntu system

> Make sure your `date`, `time` is **sync**


## Create the folder structure

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

## File to store serial number of the certificate

In my case first serial number of the certificate is `1234`
```
$ echo '1234' > serial
```

## Create a private key

```
$ openssl genrsa -aes256 -out private/cakey.pem 4096
```

Generating a private key using openssl.

`aes256` -> Encryption type

Saving the private key as `cakey.pem` inside the `private` directory.

4096 bit long key.

Provide the pass phrase.

## Root Certificate creation
We can use the private create above in the root certificate.

```
$ openssl req -new -x509 -key /root/ca/private/cakey.pem -out cacert.pem -days 3650
```
**`Note :`** You have to enter the same pass phrase of `cakey.pem` while creating the private key.

## Giving Permission to Root user only
Only root and read and write to this folder.
```
$ chmod 600 -R /root/ca
```

## Change configuration of openssl.cnf file
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

You also need the change the match to value you provided while creating the root certificate, or make it optional.

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


## Creating a request for signing our certificate.


Let say that we need to create a certificate for our webserver.

Create a private key for the webserver.
```
$ cd requests
$ openssl genrsa -aes256 -out webserver.pem 2048
```

Creating a request
```
$ openssl req -new -key webserver.pem -out webserver.csr
```

Signing the request
```
$ openssl ca -in webserver.csr -out webserver.crt
```

## Moving the file the correct folder
```
$ mv webserver.crt /root/ca/certs/
$ mv webserver.pem /root/ca/private/
```