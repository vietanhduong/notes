# PKCS#8 generate using openSSL

### Create Key pair

```console
BITS=2048
$ openssl genrsa -out keypair.pem $BITS
```

### Extract pubic key

```console
$ openssl rsa -in keypair.pem -pubout -out publickey.crt
```

### Extract private key

```console
$ openssl pkcs8 -topk8 -inform PEM -outform PEM -nocrypt -in keypair.pem -out pkcs8.key
```
