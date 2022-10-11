# Cloud Armor Notes

Cloud Armor (CA) is a security feature to protect your backend services or
buckets from DDoS, XSS, SQLi, etc. CA must be attached to a GELB (Google External
Load Balancer), External TCP LB or External SSL LB.

## Security polices

CA security polices are sets of rules that match on attributes from L3 to L7 to
protect externally facing applications or services. Each rule is evaluated with
respect to incoming traffic.

### Rule evaluation order

Rule evaliation order is determined by rule priority, from the lowest number
to the highest number.

Details: https://cloud.google.com/armor/docs/security-policy-overview#eval-order
