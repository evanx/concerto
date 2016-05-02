
## concerto

Simple git-based PKI where an API user must:
- generate a private key and self-signed public certificate e.g. using `openssl`
- create a git repo for client certs e.g. `github.com/USER/certs-concerto`
- commit the PEM file of the authorized cert to that git repo 
- provide a cert manifest file for a given API 

See example repo: `https://github.com/evanx/certs-concerto` as follows:

```shell
evans@eowyn:~/certs-concerto$ ls -C1
eowyn-evans-2016-05-01-21h38-36s.cert.pem
redishub_authorized_certs.cson
```
where the repo contains a `redishub_authorized_certs.cson` manifest:
```yaml
spec: 'concerto/manifest#0.2.0'
certs: [
  'eowyn-evans-2016-05-01-21h38-36s.cert.pem': { role: 'admin' }
]
```
where this lists active certs stored in the repo in PEM format.

We generate a cert file using `openssl` via the `concerto` bash script. We can manually check the PEM contents as follows:
```shell
evans@eowyn:~/certs-concerto$ openssl x509 -text \
  -in eowyn-evans-2016-05-01-21h38-36s.cert.pem | grep 'CN='
```
```
        Issuer: CN=eowyn-evans-2016-05-01-21h38-36s
        Subject: CN=eowyn-evans-2016-05-01-21h38-36s
```


To authenticate API access authorized by a specific Github user, the API service must know:
- the repo host and user e.g. github.com and the Github user
- the name of the certs repo e.g. `certs-concerto`
- the name of a manifest file e.g. `redishub_authorized_certs.cson`

The API service:
- provides a endpoint to import or refresh authorized certs e.g. `/importcerts` from the Github repo
- authenticates and authorizes HTTPS client requests according to the authorized certs
- entrusts Nginx to perform client cert authentication e.g. `ssl_verify_client optional_no_ca`
- the API service authorizes access according to the client cert of the HTTPS request

When the contents of the certs repo are modified:
- the user requests the service to refresh certs e.g. `/importcerts`
- the service will then sync the certs from the git repo to its own cache

Notes:
- the cert repo can be public since it contains only public certs
- if a private repo, then add the service's public ssh key as a deployment key

Benefits:
- enable secure API access via client SSL authentication
- avoid passwords and the usual email verification for lost password recovery
- the same `certs-concerto` repo could include manifest files for different API services

Costs:
- requiring the cert repo creates signup friction
- does not provide a customer messaging mechanism via verified email

The costs must be mitigated as follows:
- enable users to try the service using a less onerous token-based access scheme
- provide a utility to ease cert generation and enrollment for SSL authentication


#### Related

Bash util script: https://github.com/evanx/concerto-bash
