# Create self-signed certificates on Mac for Spring Boot project
## Install `openssl` on Mac
Install `openssl` via `homebrew`

`brew install openssl@3`

## Create root CA key and pem
`openssl genrsa -des3 -out rootca.key 4096`

`openssl req -x509 -new -nodes -key rootca.key -sha256 -days 730 -out rootca.pem`

## Create local key and csr
`openssl genrsa -out local.key 4096`

`openssl req -new -key local.key -out local.csr`

*The CN used in the certificate must match the domain name. In case of local testing, it should be localhost or any name mapped to 127.0.0.1 in /etc/hosts file.*

## Create cnf file to include domain
`nano local.cnf`

Edit the file using below content

    basicConstraints       = CA:FALSE
    keyUsage               = nonRepudiation, digitalSignature, keyEncipherment, dataEncipherment
    subjectAltName         = @alt_names
    [alt_names]
    DNS.1 = localhost

## Generate a local certificate signed by CA created above with specific expiry date

`openssl x509 -req -in local.csr -CA rootca.crt -CAkey rootca.key -CAcreateserial -out local.crt -days 365 -sha256 -extfile local.cnf`

Note that Apple has a limitation of max validity of a certificate to be 398 days. Apple recommends that certificates be issued with a maximum validity of 397 days.

Reference: https://support.apple.com/en-ca/102028

## Combine local certificate and key into a bundle p12 file

`openssl pkcs12 -export -in local.crt -inkey local.key -name local -out local.p12`

## Create Java keystore file using keytool
`keytool -importkeystore -deststorepass key-store-password -destkeystore local.jks -srckeystore local.p12 -srcstoretype pkcs12 -alias local`

## Import the local.crt into Mac keychain and set it as trusted

## Use the certificate in Spring Boot application.properties
    server.ssl.key-store=classpath:local.jks
    server.ssl.key-store-password=key-store-password
    server.ssl.key-store-type=jks
    server.ssl.key-alias=local
    server.ssl.key-password=key-password

## Enable the use of SSL in project
    server.port=8443

## Redirect all channels to the secured one

In SecurityConfig.java

    http.requiresChannel(channel -> channel.anyRequest().requiresSecure());