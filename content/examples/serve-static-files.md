---
title: "Serve static files"
draft: false
---

When serving static files, you may want Caddy to only serve static files, or sometimes you may want Caddy to serve static files and run as a reverse proxy.

## _Only_ serve static files

To configure Caddy to _only_ serve static files, use the `file_server` directive, then set the `root` directory. The `*` immediately after `root` tells Caddy that it should match _all_ requests&ndash;without it, Caddy wouldn't work.

```Caddyfile
localhost:2015 {
    root * /srv/static-files
    file_server
}
```
