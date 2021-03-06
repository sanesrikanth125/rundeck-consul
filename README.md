# rundeck-consul

A simple Python Bottle app that presents Consul nodes and services as RunDeck
resource and option models.

**THIS IS A WORK IN PROGRESS!**
Currently I'm only using it internally, and it has not been throughly tested.

## Requirements
* bottle
* python-consul

## Endpoints
* `/resource` - Returns a Rundeck [resource-json-v10][1] formatted resource
  model of Consul nodes and services.
* `/services` - Returns a simple [json-formatted][2] list of Consul services
  for use as a Rundeck option model provider. The following optional parameters are
  allowed (and multiple can be used together):
  * `tag` - return services that have a given tag
  * `tags` - return all services matching (comma-separated) list of tags
  * `dc` - return services in a given datacenter
  * `startswith` - return services that start with given string
  * `contains` - return services that contain the given string
  * `endswith` - only return services that contain a given string
  * `regex` - return services matching regex pattern (accepts URL-encoded string)

##### Project-specific endpoints
Optional projects can be configured (for example multiple consul clusters
or environments):
* `/resource/<project>`
* `/services/<project>`

## Running
Run `./app.py`, or if using a configuration file (see below),
`./app.py --config /path/to/config.json`. How to keep it running is up to you.

## Configuration File
The configuration file is optional. If provided it should follow the format
indicated below. If not provided the app will listen on `0.0.0.0:8080` and
will attempt to connect to consul at `127.0.0.1:8500`.

* `host` - listen address for server (default: `0.0.0.0`)
* `port` - listen port for server (default: `8080`)
* `consul` - dictionary of the following Consul connection parameters:
  * `host` - consul host (default: `127.0.0.1`)
  * `port` - consul host (default: `8500`)
  * `token` - consul token (default: `None`)
  * `scheme` - consul HTTP(S) connection scheme (default: `http`)
  * `verify` - whether to verify TLS cert (default: `True`)
  * `cert` - consul TLS certificate (default: `None`)
* `datacenters` - only include certain datacenters in resource
* `services` - only include certain services in resource
* `exclude` - return all services except for these
* `node_attributes` - map of additional attributes to map to all nodes returned
  in the resource model (see below for examples)
* `append_tags` - whether to append Consul tags to their respective service,
  or to associate them directly to the node (default is `false`).  For example,
  given a Consul node with service `mysql` & tag `master`, if `append_tags` is
  true the node will have an attribute of `mysql:master`... or `mysql` and `master` if false.

If `datacenters`, `services`, and `exclude` are not provided, queries to
`/resource` (or `/resource/<project>`) will return all services in all datacenters.

Example (not using multiple "projects"):
```json
{
  "host": "0.0.0.0",
  "port": 8080,
  "consul": {
    "host": "localhost",
    "port": "8500",
    "token": "optionalsecret"
  },
  "datacenters": ["dc1", "dc2"],
  "services": ["service1", "service2"],
  "exclude": ["excluded_service1"],
  "node_attributes": {
    "username": "ubuntu"
  },
  "append_tags": true
}
```

Example (multiple environments/clusters):
```json
{
  "host": "0.0.0.0",
  "port": 8080,
  "projects": {
    "dev": {
      "consul": {
        "host": "consul-dev.example.net",
        "token": "devsecret"
      },
      "node_attributes": {
        "username": "ubuntu",
        "environment": "dev"
      }
    },
    "prod": {
      "consul": {
        "host": "consul-prod.example.net",
        "token": "prodsecret"
      },
      "node_attributes": {
        "username": "ubuntu",
        "environment": "prod"
      }
    }
  }
}
```

[1]: http://rundeck.org/docs/man5/resource-json.html
[2]: http://rundeck.org/docs/manual/jobs.html#json-format
