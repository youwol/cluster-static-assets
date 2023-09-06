# Cluster static assets data

This repository provide data for the static assets of Youwol cluster.

Static assets are served from an nginx official Docker image, and that repository allow customization of that
instance of nginx.

## Repository structure

### [templates](./templates)

The templates directory, to be mounted into the docker image directory `/etc/nginx/templates`.
Basically environment variables in the `*.template` files will be expanded and the resulting files will be put in the
Docker image directory `/etc/nginx/conf.d` (see [Official
documentation](https://hub.docker.com/_/nginx/)).

#### Environment variables

* `PLATFORM_HOST` is used for defining the full URI redirection for the default route.

### [sites](./sites)

The content directory, to be mounted into the Docker image directory `/usr/share/nginx`

## Virtual Hosts

Since all resources are served from the same Docker image, virtual hosts are used to distinguish the various usages.

These virtual hosts will be used for define the corresponding Kubernetes services & ingresses −
see [Deployment](#deployment)
for details.

* [config for host `static-assets`](./templates/static-assets.conf.template) serve the static assets content from
  [sites/static-assets](./sites/static-assets)
* [config for host `default-route`](./templates/default-route.conf.template) redirect everything to 
  http://_PLATFORM_HOST_/application/@youwol/platform/latest (see [below](#environment-variables))
* [config for host `maintenance`](./templates/maintenance.conf.template) serve only one page, with HTTP status 503.
  This page will include content from [sites/includes/maintenance/details.txt](./sites/includes/maintenance/details.txt)
* [config for any other hosts](./templates/default.conf.template) always respond with error 404.

The [common config](./templates/common.inc.template) define the same errors pages for status 403 & 404 (from
[sites/errors](./sites/errors)) and the same favicon (
from [sites/static-assets](./sites/static-assets/favicon_64x64.png)) for all
these hosts.

## Testing

Here’s a basic example of how to run locally an nginx Docker image with this repository, binding nginx port to port 
8080 on a loopback IP − try not to blindly copy it in your terminal:

```shell
docker run \
  --name testing-static-assets \
  --rm \
  --publish 127.x.x.x:8080:80 \
  -e PLATFORM_HOST="example.com" \
  -v ./templates:/etc/nginx/templates \
  -v ./sites:/usr/share/nginx \
  nginx
```

For testing the various virtual hosts, either use appropriate parameters when using CLI tools (i.e. `--header 'Host:
maintenance'`) or add relevant entries in local `/etc/hosts` to allow resolution of these hosts to the loopback IP:

```text
# in /etc/hosts
[…]
# Use the same ip as for publishing the ports of the docker image
127.x.x.x maintenance default-route static-assets
[…]
```

With that example, the various virtual hosts can be access from a browser (i.e. http://maintenance:8080/).

## Deployment

See documentation in [k8s-deployments repository](https://github.com/youwol/k8s-deployments/tree/nevado-tres-cruces/infra/static-assets)
