
## concerto

Simple git-based PKI where an API user must:
- generate a private key and self-signed public certificate e.g. using `openssl`
- create a git repo for client certs e.g. `github.com/USER/certs-concerto`
- commit the PEM file of the authorized cert to that git repo 
- provide a cert manifest file for a given API 

<img src="https://evanx.github.io/images/rquery/concerto-repo.png">

See example repo: `https://github.com/evanx/certs-concerto` as follows:

Rhe repo contains a `redishub_authorized_certs.cson` manifest:
```yaml
spec: 'concerto/manifest#0.2.0'
certs: [
  'eowyn-evans-2016-05-01-21h38-36s.cert.pem'
]
```
where this lists active certs stored in the repo in PEM format. 

We generate a cert file using `openssl rsa` via the `concerto` bash script. 
```shell
evans@eowyn:~/concerto$ bin/concerto gen
```
```
Generating a 2048 bit RSA private key
writing new private key to 'eowyn-evans-2016-05-02-04h55-22s.privkey.pem'
        Issuer: CN=eowyn-evans-2016-05-02-04h55-22s
        Subject: CN=eowyn-evans-2016-05-02-04h55-22s
```
where the issuer and subject are the same since it is self-signed. 

Note that a default cert `id` is chosen according the hostname, user and timestamp. This `CN` is defaulted to this `id.` Since we might prefer this information to be private. Therefore private repos must be supported, with access granted to the service's ssh key. 

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
- enables secure API access via client SSL authentication
- avoids passwords and the usual email verification for lost password recovery
- enables a minimal service implementation with secure access
- the same `certs-concerto` repo could include manifest files for different API services

Costs:
- requiring the cert repo creates signup friction
- does not support a customer notification mechanism via verified email
- requires a Github account 

The costs must be mitigated as follows:
- enable users to try the service using a simpler token-based access scheme
- provide a utility to ease cert generation and enrollment for SSL authentication
- support HTTP-based customer notification e.g. via Slack, Telegram

In the future, we will implement a cert registry service to replace the git-based repo:
- enable an account to be registered with auto-enrollment of the original client cert
- add/update certs to the registry via HTTP POST with an authorised "admin" cert


#### Related

Bash util script: https://github.com/evanx/concerto-bash
