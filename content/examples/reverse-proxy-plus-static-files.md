---
title: "Reverse proxy plus static files"
draft: false
---

If you're deploying an app built with a framework like Django, you'll likely want to serve the application through a reverse proxy, then have Caddy serve your static files directly. However, if you just try to put a `root` and `file_server` directive into a site block that contains a reverse proxy, it won't work. As Francis Lavoie explains on the [Caddy Forum](https://caddy.community/t/django-static-assets-not-getting-served-in-caddy-v2/8345/2):

> The issue is that the `reverse_proxy` is handling all requests before it reaches `file_server`. Caddy doesn’t know otherwise which requests should avoid using the proxy and which should be handled via the file server.
>
> When using the Caddyfile, directives are handled in a specific order, described in the following docs page. Specifically, `reverse_proxy` is ordered higher than `file_server`, so it’ll get handled first if a request matched by it (most directives match _all requests_ by default).

Thankfully, Caddy has features to override this ordering.

## Recommended approach

Caddy includes a `handle` and `handle_path` directive. These will apply rules in order from the first matching `handle`.

The `handle_path` directive works like the `handle` directive, but it strips the matched path prefix.

```Caddyfile
localhost:2015 {
    handle_path /static/* {
        root * /app/srv/static
        file_server
    }
    handle {
        reverse_proxy localhost:8000
    }
}
```

## Alternative approach

**I don't recommend this approach,** but it's more-or-less what Lavoie recommended in the forum, and I've seen the general approach taken in many Caddyfile examples. It feels imperative and clunky compared to the `handle`-based approach.

```Caddyfile
localhost:2015 {
    root * /srv/app
    file_server /static/*

    @notStatic {
        not path /static/*
    }

    reverse_proxy @notStatic localhost:8000
}
```
