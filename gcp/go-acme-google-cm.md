# go-acme (lego) + Google Certificate Manager

Order a public CA from google using ACME.

This required a EAB account is requestd from Google. (*eab: External Account Binding).

Permissions required:
* `roles/publicca.externalAccountKeyCreator`

Enable the Public CA API:

```console
$ gcloud services enable publicca.googleapis.com
```

To do that:
```console
$ gcloud beta publicca external-account-keys create

b64MacKey: ***
keyId: ***
```

Save 2 keys, we will use them later.

### Lego

```go
package main

import (
	"crypto"
	"crypto/rsa"
	"crypto/x509"
	"encoding/pem"
	"fmt"
	"log"

	"github.com/go-acme/lego/v4/certcrypto"
	"github.com/go-acme/lego/v4/certificate"
	"github.com/go-acme/lego/v4/lego"
	"github.com/go-acme/lego/v4/providers/dns/cloudflare"
	"github.com/go-acme/lego/v4/registration"
)

type User struct {
	Email        string
	Registration *registration.Resource
	key          crypto.PrivateKey
}

func (i *User) GetEmail() string {
	return i.Email
}

func (i *User) GetRegistration() *registration.Resource {
	return i.Registration
}

func (i *User) GetPrivateKey() crypto.PrivateKey {
	return i.key
}

// privKey RSA private key, this key will be used for attach to an user
const privKey = ``

func OrderCertificate() {
	block, _ := pem.Decode([]byte(privKey))
	pk, err := x509.ParsePKCS8PrivateKey(block.Bytes)
	if err != nil {
		log.Fatal(err)
	}

	privateKey := pk.(*rsa.PrivateKey)

	myUser := &User{
		Email: "your_email",
		key:   privateKey,
	}

	config := lego.NewConfig(myUser)

	// This CA URL is configured for a local dev instance of Boulder running in Docker in a VM.
	config.CADirURL = "https://dv.acme-v02.api.pki.goog/directory" // google acme server (prod)
	config.Certificate.KeyType = certcrypto.RSA2048

	// A client facilitates communication with the CA server.
	client, err := lego.NewClient(config)
	if err != nil {
		log.Fatal(err)
	}

	var cf *cloudflare.DNSProvider
	if cf, err = NewCloudflareProvider(); err != nil {
		log.Fatal(err)
	}

	err = client.Challenge.SetDNS01Provider(cf)
	if err != nil {
		log.Fatal(err)
	}

	// New users will need to register
	//reg, err := client.Registration.Register(registration.RegisterOptions{TermsOfServiceAgreed: true})
	reg, err := client.Registration.RegisterWithExternalAccountBinding(registration.RegisterEABOptions{
		TermsOfServiceAgreed: true,
		Kid:                  "key_id",
		HmacEncoded:          "b64_mac_key",
	})
	if err != nil {
		log.Fatal(err)
	}
	myUser.Registration = reg

	request := certificate.ObtainRequest{
		Domains: []string{"*.local.kyberengineering.io"},
		Bundle:  true,
	}
	certificates, err := client.Certificate.Obtain(request)
	if err != nil {
		log.Fatal(err)
	}

	// Each certificate comes back with the cert bytes, the bytes of the client's
	// private key, and a certificate URL. SAVE THESE TO DISK.
	fmt.Printf("%#v\n", certificates)
}
```

## References
* https://www.rfc-editor.org/rfc/rfc8555#section-7.3.4
* https://go-acme.github.io/lego/usage/library/
* https://cloud.google.com/certificate-manager/docs/public-ca-tutorial
