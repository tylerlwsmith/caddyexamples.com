---
title: "Prevent automatic http-to-https redirect"
draft: false
---

Occasionally while developing a web app locally, you need to build an isolated feature that _requires_ `https` (such as a [service worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers#setting_up_to_play_with_service_workers)). Using a self-signed certificate locally introduces the annoyance of Chrome returning an error every few hours that says `Your connection is not private`.

It would be nice to only deal with those self-signed `https` warnings when you absolutely needed to use `https`, then access the site warning-free via `http` the rest of the time. However, Caddy automatically adds an `http`-to-`https` redirect for all domain [site addresses](https://caddyserver.com/docs/caddyfile/concepts#addresses).

You can prevent the automatic redirection from `http`-to-`https` by listing both `http` and `https` addresses at the beginning of the [site block](https://caddyserver.com/docs/caddyfile/concepts#blocks):

```Caddyfile
http://mysite.test, https://mysite.test {
	# bind allows access to containers from host when running Caddy in Docker
	bind 0.0.0.0

	# Issue a self-signed certificate for development
	tls internal

	respond "Hello, world! I am being accessed from {scheme}."
}
```

**Do not use this configuration in production.** There are very few responsible reasons to serve an `http` version of your production site. Only use this configuration for development.

## Gotchas

When developing locally, you may need to add the domains to your system's `hosts` file if they aren't already there.
