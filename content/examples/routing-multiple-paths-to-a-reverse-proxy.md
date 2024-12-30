---
title: "Routing multiple paths to a reverse proxy"
draft: false
---

Sometimes your app will need to route a handful of paths to one service and all other paths to another. Caddy's [named matchers](https://caddyserver.com/docs/caddyfile/matchers#named-matchers) allow you to define a set of [path directives](https://caddyserver.com/docs/caddyfile/matchers#path) then route them all to a single reverse proxy.

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

You may wonder why we're matching most `path` directives against two different routes, using `path /posts /posts/*` instead of `path /posts*`.

Using `path /posts*` could include paths that you don't intend to proxy. If we used `path /posts*` and `path /static*`, we would also match on the following routes:

- `/postscript`
- `/posts-authors`
- `/static-electricity`
- `/static-site-generators`

This may be the desired behavior in your application, but it's probably not.
