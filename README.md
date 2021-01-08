# PKI
A repo holding the files to create a Public Key Infrastructure

## Create the root CA private key and public certificate
Locate your `openssl.cnf` file with for example `locate openssl.cnf` and copy it to your working folder

If the [ usr_cert ] and [ server_cert ] sections are missing from `openssl.cnf`, add them
following this example:
```
[ usr_cert ]
# Extensions for client certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection

[ server_cert ]
# Extensions for server certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
```

Create the `private/`, `certs/`, and `newcerts` folders if they are not present (check that these folders name correspond to the ones in the `openssl.cnf` file. Create also the empty
`index.txt` file for your CA database in the working folder, and the `serial` file. Then add
`1000` to the first line in the `serial` file, for example with `echo 1000 > serial`

### Create the root CA private key and make the key readable only by owner
`openssl req -config openssl.cnf -newkey rsa:2048 -keyout ./private/cakey.pem -noout`

`chmod 400 private/ca.key.pem`

### Create the self signed CA certificate
`openssl req -config openssl.cnf -key ./cakey.pem -new -x509 -days 1095 -sha256 -out ./cacert.pem`

### Verify the root CA certificate
`openssl x509 -noout -text -in ./cacert.pem`

## Generate the private key and signed certificate for a server

Private key:

`openssl genrsa -out ./private/server.key.pem`

Certificate signing request:

`openssl req -config openssl.cnf -key ./private/server.key.pem -new -sha256 -out ./certs/server.csr.pem`

Sign the server certificate:

`openssl ca -config openssl.cnf -extensions server_cert -days 1045 -notext -md sha256 -in certs/server.csr.pem -out certs/server.cert.pem -create_serial`

Verify the server certificate:

`openssl x509 -noout -text -in ./certs/server.cert.pem`

## Generate the private key and signed certificate for the client

Private key:
 `openssl genrsa -out ./private/client.key.pem`

Certificate signing request:

`openssl req -config openssl.cnf -key client/client.key.pem -new -sha256 -out
client/client.csr.pem`

Sign the client certificate:

`openssl ca -config openssl.cnf -extensions usr cert -days 375 -notext -md sha256 -in client/client.csr.pem -out client/client.cert.pem`
