
# concerto

Simple git-based PKI with goals:
- enable a minimal secure API implementation
- use self-signed client SSL certs for authentication


### User story

Say you want to build and deploy a cloud-based API, without any bells and whistles. The initial requirement for the API is that you can use it yourself to build apps. You probably want to show it to your friends and colleagues, and let them "signup" to try it out and give you some feedback.

Months later, you might decide that you want to launch it professionally by building a website with signup, OAuth, email verification, password recovery etc.

With Concerto you can avoid that overhead initially, to build a "minimum viable" API implementation, that most importantly doesn't compromise on security. It does create some enrollment friction e.g. creating client certs and committing them to an authorized certs repo. But for yourself and hopefully some other technical consumers, that's not a show-stopper.

An API user must:
- generate a private key and self-signed public certificate e.g. using `openssl`
- create a git repo for client certs e.g. `github.com/GHUSER/certs-concerto`
- commit the PEM file of authorized certs to that git repo
- provide a manifest file for a given API e.g. `redishub/manifest.json`
- in the manifest, specify a role-based access permissions e.g. the `admin` role has full access
- provide a list of certs for each role e.g. `admin.txt`


### Certs repo

<img src="https://evanx.github.io/images/rquery/concerto-repo.png">

See example repo: https://github.com/evanx/certs-concerto

The repo contains a manifest for my `redishub` service:
```json
{
   "spec": "redishub/manifest#0.3.0",
   "permissions": {
      "admin": [
         {
            "resource": {
               "regexp": ".*"
            },
            "allow": [
               "all-actions"
            ]
         }
      ]
   }
}
```
where this declares permissions for roles, e.g. for `admin` role.

The certs with the admin role are listed in `admin.txt` as follows:
```
eowyn-evans-2016-05-01-21h38-36s.cert.pem
eowyn-evans-2016-05-05-18h34-56s.cert.pem
```
where these certs are found in the `certs/` folder of the repo.

Find an empty sample manifest for my `redishub.com` service at:
https://github.com/evanx/concerto/blob/master/sample/redishub/manifest.json

### Cert gen

We generate a cert file using `openssl` e.g. via the `concerto` bash script:
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

See the `bin/concerto` bash util script:
https://github.com/evanx/concerto/blob/master/bin/concerto

<img src="https://evanx.github.io/images/rquery/concerto-help.png">


### API integration

The API service:
- provides a endpoint to sync authorized certs from the repo e.g. `/certs-sync`
- authenticates and authorizes HTTPS client requests according to the authorized certs
- entrusts Nginx to perform client cert authentication e.g. `ssl_verify_client optional_no_ca`
- the API service authorizes access according to the client cert of the HTTPS request

When the contents of the certs repo are modified:
- the user requests the service to refresh certs e.g. `/certs-sync`
- the service will then sync the certs from the git repo to its own cache


### Technical notes

- the cert repo can be public since it contains only public certs
- if a private repo, then add the service's public ssh key as a deployment key


### Cost/benefit

Benefits:
- enables secure API access via client SSL authentication
- avoids passwords and the usual email verification for lost password recovery
- enables a minimal service implementation with secure access
- the same `certs-concerto` repo could include manifest files for different API services
- the repo provides transparency and versioning of authorized certs

Costs:
- requiring the cert repo creates signup friction
- does not support a customer notification mechanism via verified email
- requires a Github account

The costs should be mitigated as follows:
- enable users to try the service using a simpler token-based access scheme
- provide a utility to ease cert generation and enrollment for SSL authentication
- support HTTP-based customer notification e.g. via Slack, Telegram


### TODO

- support private repos

### Future plans

We wish to implement a cert registry service using Redis to:
- enable an account to be registered with auto-enrollment of the original client cert
- add/update certs to the registry via HTTP POST with an authorised "admin" cert

For earlier work along these lines, see:
https://github.com/evanx/certserver

#### Related

https://github.com/evanx/rquery
