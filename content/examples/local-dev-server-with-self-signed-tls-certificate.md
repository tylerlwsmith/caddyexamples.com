---
title: "Local dev server with self-signed TLS certificate"
draft: false
---

Some applications require HTTPS to function correctly, or you may prefer to develop with HTTPS
to more closely match your production environment.

In this situation, Caddy can be useful as a reverse-proxy in front of your local dev server.
If you choose a domain that Caddy recognizes as localhost, it will automatically sign the
certificate with its internal CA.

However, if you need a certificate for a specific domain, it might fail with the following error:

```log
tls.issuance.acme.acme_client    challenge failed
```

In that case, you can force Caddy to use a self-issued certificate with the
[tls internal](https://caddyserver.com/docs/caddyfile/directives/tls) directive:

```Caddyfile
local-alias.my-company.com {
	reverse_proxy * http://localhost:1234
	tls internal {
		on_demand
	}
}
```
