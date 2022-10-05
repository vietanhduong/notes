# Google Certificate Manager notes

### Resources
* Certificate: Google certificate
* DNS Authorization: Used for verify domains. DNS Authorization is required when certificate's domain is wildcard.
* Certificate Map: Group of certificate
* Certiciate Map Entry: Use to link certificates to a map

### Notes
* Certificate (google) just work with Google LB or Gateway, ... some services of Google.
* TLS will be teared down on their service. E.g: If you have an VM and you deploy a `nginx` on the vm.
If you using Google Certificated Manager, you don't need manage the certificate (cert, private key).
All you need is attach a certificate map into your LB and forward the request to your VM.

### References
* https://cloud.google.com/kubernetes-engine/docs/how-to/secure-gateway
* https://cloud.google.com/certificate-manager/docs/deploy-google-managed-lb-auth
