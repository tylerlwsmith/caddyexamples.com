---
title: "Using an environment variable for the hostname with a www redirect and localhost fallback"
draft: false
---

Ideally, a single Caddyfile could handle the following use cases without relying on different files for development and production:

- When running locally, develop using the `localhost` hostname without having to provision certificates.
- When running in production, automatically provision TLS certificates for the appropriate hostnames.
- Redirect `www` to non-`www` or vice versa.

Using an environment variable in the [site address](https://caddyserver.com/docs/caddyfile/concepts#addresses) allows Caddy to support all of these use cases with a single Caddyfile.

```Caddyfile
{$SITE_HOSTNAME:localhost:80} {
	# bind allows access to containers from host when running Caddy in Docker
	bind 0.0.0.0

	respond "Hello, world!"
}

# Handle redirects from www to non-www
www.{$SITE_HOSTNAME:localhost:80} {
	# bind allows access to containers from host when running Caddy in Docker
	bind 0.0.0.0

	redir {scheme}://{$SITE_HOSTNAME:localhost:80}{uri}
}
```

During development, no value is needed for the `$SITE_HOSTNAME` environment variable: it will default to `localhost:80`. Because the default `http` port `:80` is specified, Caddy will serve the site strictly over `http` without attempting to provision a certificate.

In production, the `$SITE_HOSTNAME` environment variable should be set to the appropriate domain (example: `yoursite.com`). Caddy will then automatically provision the TLS certificates, allowing you to run the same Caddyfile in both development and production.

The `{scheme}` [placeholder](https://caddyserver.com/docs/caddyfile/concepts#placeholders) will ensure that if the request used `http` (like `http://www.locahost`), it will redirect to its non-`www` counterpart with the correct scheme.

If you instead wanted to redirect from non-`www` to `www`, you could use the following configuration.

```Caddyfile
www.{$SITE_HOSTNAME:localhost:80} {
	# bind allows access to containers from host when running Caddy in Docker
	bind 0.0.0.0

	respond "Hello, world!"
}

# Handle redirects from non-www to www
{$SITE_HOSTNAME:localhost:80} {
	# bind allows access to containers from host when running Caddy in Docker
	bind 0.0.0.0

	redir {scheme}://www.{$SITE_HOSTNAME:localhost:80}{uri}
}
```

## Gotchas

There are a few things you need to watch out for if you use this configuration.

**The `www` to non-`www` redirect won't work correctly if you're mapping to a different port on the host when running inside a container.** If Caddy is running on port `:80` in a container but it is mapped to `:8080` on the host, the example above would redirect `http://www.localhost:8080` to `http://localhost:80`. This is because Docker maps the host port `:8080` to `:80` before Caddy ever sees it, so inside the container Caddy is actually redirecting `http://www.localhost:80` to `http://localhost:80`.

**You can't use the environment variable `$HOSTNAME` if you're running inside of a Docker container.** The Caddy container already has an environment variable called `$HOSTNAME`, and it cannot be overwritten.

**Testing the `www.localhost` redirect may require changes to your computer's `hosts` file.** The `www.localhost` address worked fine on my MacBook with no additional configuration, but your mileage may vary.
