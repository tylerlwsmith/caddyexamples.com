---
title: "Return a plain text response"
draft: false
---

Returning a plain text response with Caddy can be useful for debugging, ensuring that the server is indeed working before configuring a something like a reverse proxy.

## Example

```Caddyfile
localhost:8080 {
    respond "Hello, world!"
}
```
