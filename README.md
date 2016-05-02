
## concerto

Simple git-based PKI where an API user must:
- create a git repo for client certs e.g. `github.com/USER/certs-concerto`
- use `openssl` to generate a private key and self-signed public certificate
- commit authorized certs to git repo e.g. `certs-concerto`
- provide a manifest file of active certs e.g. `SERVICE_authorized_certs.cson`

To authenticate access, API service is configured with:
- the repo host e.g. github.com
- the user name on the repo host e.g. your Github user
- a well-known name of the certs repo e.g. `certs-concerto`
- a well-known manifest name of authorized certs e.g. `redishub_authorized_certs.cson`

The API service:
- provides a endpoint to import or refresh authorized certs e.g. `/importcerts`
- otherwise the service authenticates HTTPS requests according to the committed certs

When the contents of the certs repo are modified:
- the user requests the service to import/refresh certs e.g. `/importcerts`
- the service will then sync the certs from the git repo to its own cache

Notes:
- the cert repo can be public since it contains only public certs
- if a private repo, then add the service's public ssh key as a deployment key
- enables API signup and access without requiring email verification or passwords
- the same cert repo could be shared by multiple services

Costs:
- 

#### Example

See example repo: `https://github.com/evanx/certs-concerto` as follows:

```shell
evans@eowyn:~/certs-concerto$ ls -C1
eowyn-evans-2016-05-01-21h38-36s.cert.pem
redishub_authorized_certs.cson
```
where the repo contains a `redishub_authorized_certs.cson` manifest:
```shell
evans@eowyn:~/certs-concerto$ cat redishub_authorized_certs.cson 
spec: 'concerto/manifest#0.2.0'
certs: [
  'eowyn-evans-2016-05-01-21h38-36s.cert.pem'
]
```
where this lists active cert PEM files in the repo:
```shell
evans@eowyn:~/certs-concerto$ openssl x509 -text \
  -in eowyn-evans-2016-05-01-21h38-36s.cert.pem | grep 'CN='
        Issuer: CN=eowyn-evans-2016-05-01-21h38-36s
        Subject: CN=eowyn-evans-2016-05-01-21h38-36s
```

#### Related

Bash util script: https://github.com/evanx/concerto-bash
