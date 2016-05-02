
## concerto

A light-weight API authentication scheme where an API user must:
- create a git repo for client certs e.g. `github.com/USER/certs-concerto`
- use `openssl` to generate a private key and self-signed public certificate
- commit authorized certs to git repo e.g. `certs-concerto`
- provide a manifest file of active certs e.g. `SERVICE_authorized_certs.cson`
- the service provides a endpoint to import or refresh authorized certs e.g. `/importcerts`
- otherwise the service authenticates HTTPS requests according to the authorized certs

To authenticate access, API service is configured with:
- the repo host e.g. github.com
- the user name on the repo host e.g. your Github user
- a well-known name of the certs repo e.g. `certs-concerto`
- a well-known name of the authorized certs e.g. `redishub_authorized_certs.cson`

When the contents of the certs repo are modified:
- a webhook endpoint provided by the service is invoked e.g. `/importcerts`
- the service will then sync the certs

Requests are authenticated


#### Example

See example repo: https://github.com/evanx/certs-concerto - containing `redishub_authorized_certs.cson` manifest, listing the PEM files in the repo.


#### Related

Bash util script: https://github.com/evanx/concerto-bash
