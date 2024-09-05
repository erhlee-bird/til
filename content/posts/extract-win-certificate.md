---
date: '2024-09-05T09:39:33-04:00'
draft: false
params:
  author: "Eric Lee"
tags: ["bash", "pefile"]
title: 'Extract Win Certificate'
---

Breaking it down,

1. use 7z to extract the WIN_CERTIFICATE
2. skip metadata fields and extract cert bytes
3. parse certificate details with openssl
4. additionally grab Validity period end
5. remove leading and trailing whitespace
6. reverse line order to place "Not After" after "Subject"
7. convert output to jsonl
8. test that output is queryable with jq

```shell
7z e -so foo.exe CERTIFICATE | \
dd bs=1 skip=8 status=none | \
openssl pkcs7 -inform der -print_certs -noout -text | \
grep -E '(Subject:|Not After)' | \
awk '{gsub(/^[ \t]+|[ \t]+$/, "", $0); print $0}' | \
awk '{if (NR%2==1) {line1=$0} else {print $0; print line1}}' | \
awk -F': ' '/Subject/ {subj=$2} /Not After/ {print "{\"Subject\": \"" subj "\", \"Not After\": \"" $2 "\"}"}' | \
jq
```

Output then looks like the following:

```
{
  "Subject": "C=US, O=DigiCert Inc, OU=www.digicert.com, CN=DigiCert Trusted Root G4",
  "Not After": "Nov  9 23:59:59 2031 GMT"
}
{
  "Subject": "C=US, O=DigiCert, Inc., CN=DigiCert Trusted G4 RSA4096 SHA256 TimeStamping CA",
  "Not After": "Mar 22 23:59:59 2037 GMT"
}
{
  "Subject": "C=US, O=DigiCert, Inc., CN=DigiCert Trusted G4 Code Signing RSA4096 SHA384 2021 CA1",
  "Not After": "Apr 28 23:59:59 2036 GMT"
}
{
  "Subject": "C=US, O=DigiCert, Inc., CN=DigiCert Timestamp 2023",
  "Not After": "Oct 13 23:59:59 2034 GMT"
}
```
