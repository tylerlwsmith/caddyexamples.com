---
title: "Routing multiple paths to a reverse proxy"
draft: false
---

Sometimes your app will need to route a handful of paths to one service and all other paths to another. Caddy's [named matchers](https://caddyserver.com/docs/caddyfile/matchers#named-matchers) allow you to define a set of [path directives](https://caddyserver.com/docs/caddyfile/matchers#path) then route them all to a single reverse proxy/

```Caddyfile
:80 {
	bind 0.0.0.0
	encode zstd gzip
	@webapp {
		path /
		path /posts /posts/*
		path /tags /tags/*
		path /static /static/*
	}

	handle @webapp {
		reverse_proxy webapp:3000
	}
	handle {
		reverse_proxy wordpress:80
	}
}
```

Alternatively, you can omit the [handle](https://caddyserver.com/docs/caddyfile/directives/handle) blocks:

```Caddyfile
:80 {
	bind 0.0.0.0
	encode zstd gzip
	@webapp {
		path /
		path /posts /posts/*
		path /tags /tags/*
		path /static /static/*
	}

	reverse_proxy @webapp webapp:3000
	reverse_proxy wordpress:80
}
```
