---
title: "Binding to 0.0.0.0 to expose Caddy service in Docker"
draft: false
---

Docker services that you want exposed to outside of the container can't be resolved to localhost, they must instead be resolved down to `0.0.0.0` so services outside of the container can access them. However, if you set your domain as `0.0.0.0:80`, you will see the following error:

<!-- Include the error message so people land on this page from Google -->

Because of this, you must use the `bind` directive.

```Caddyfile
:80 {
    bind 0.0.0.0
    respond "Hello, world!"
}
```
