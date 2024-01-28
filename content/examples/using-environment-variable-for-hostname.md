---
title: "Using an environment variable for the hostname with a www redirect and localhost fallback"
draft: false
---

Setting a site's hostname as an environment variable in a Caddyfile enables the usage of non-https `localhost` when developing locally, then switching to a proper domain name in production while reaping the benefits of Caddy's automatic TLS certificates.

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

During development, you could run this without providing the `SITE_HOSTNAME` variable and it would default to `localhost`. If you set `SITE_HOSTNAME` to `example.com` when running in production, Caddy would automatically provision the TLS certificates, allowing you to run the same Caddyfile in development and production.

## Gotchas

**The redirect won't work correctly if you're mapping to a different port on the host than the port running inside a container.** If Caddy is running on port `:80` in a container but it is mapped to `:8080` on the host, the example above would redirect `http://www.localhost:8080` to `http://localhost:80`. This is because Docker changes port `:8080` to `:80` before Caddy ever sees it, so in a container Caddy is actually redirecting `http://www.localhost` to `http://localhost` (port `:80` is dropped because it is the default http port).

**You can't use `HOSTNAME` for the environment variable inside the container.** The Caddy container already has an environment variable called `HOSTNAME`, and it cannot be overwritten.

**Testing the `www.localhost` redirect may require changes to your computer's `hosts` file.** The `www.localhost` address worked fine on my MacBook with no configuration, but your mileage may vary.
