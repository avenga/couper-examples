# Proxy API Requests

Most modern Web applications use APIs to get information for display, to
trigger actions or to check if a user is authorized to see or do
stuff.

Those APIs may be part of your application (i.e. you have written it)
or a third-party service running elsewhere. 

Wherever they run – our favorite spot for them is behind Couper :)

Let's create a Couper configuration that exposes two backend services
in a consolidated API for the client.

## Configuration

A basic gateway configuration defines upstream backend services and
"mounts" them on local API endpoints:

```hcl
server "my-api" {
  api {
    endpoint "/example/**" {
      path = "/api/v2/**"
      backend {
        origin = "http://myservice:8080"
      }
    }
  }
}
```

### Routing

So what happens when you request `localhost:8080/example`?

The `api` block is implicitly mounted on `/` on the exposed server
(if no `base_path` is given). Therefore `/example` is matched against
the defined `endpoint` configurations. We only have one here, that
uses the "catch-all" operator `/**`. This means that our `endpoint`
handles `/example` (yes, without the trailing `/`), `/example/` and
every possible sub-path of it, such as `/example/user/login`.

The endpoint defines a `backend` to proxy requests to. The only
mandatory information is the `origin` attribute: Where should Couper
connect to? This could be a local application (e.g. running in the same
Kubernetes namespace), or some remote service. We can define a lot of
HTTP related settings here.

### Path Mapping

We have decided to mount `myservice` onto the `/example` path. But
the origin doesn't know about that path. Its endpoints live under
`/api/v2`. We need to remove our local path prefix and adapt it to
the backend base prefix.

The `path` property does just that. Everything that was matched by
the `/**` operator of our endpoint is added to the new path where we
use the operator again. `/example/user/login` becomes
`http://myservice:8080/api/v1/user/login` in the backend request.
