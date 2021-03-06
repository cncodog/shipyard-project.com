+++
Categories = []
Description = ""
Tags = []
title = "Automated Deployment"
date = 2014-09-20T03:30:04Z
+++

This will deploy Shipyard using the current Docker host.

Once deployed, the script will output the URL to connect along with credential
information.

```bash
curl -sSL https://shipyard-project.com/deploy | bash -s
```

For full usage details:

```bash
curl -sSL https://shipyard-project.com/deploy | bash -s -- -h
```

```bash
Shipyard Deploy uses the following environment variables:
  ACTION: this is the action to use (deploy, upgrade, node, remove)
  DISCOVERY: discovery system used in Swarm (only valid with the 'node' action)
  IMAGE: this overrides the default Shipyard image
  PREFIX: prefix for container names
  SHIPYARD_ARGS: these are passed to the Shipyard controller container as controller args
  TLS_CERT_PATH: path to certs to enable TLS for Shipyard
  PORT: listen port for the shipyard controller (default: 8080)
  PROXY_PORT: port for the docker proxy (default: 2375)
```

Note: all variables are optional.

## Options

To customize the deployment, you can specify the following environment
variables.

### ACTION

This controls the action for the deployment.  Available options are:

- `deploy`: Deploy a new Shipyard instance
- `upgrade`: Upgrade an existing instance (note: you will need to pass the
same environment variables as when you deployed to keep the same configuration)
- `node`: Add current Docker engine as a new Swarm node in the cluster
- `remove`: Completely removes Shipyard

### IMAGE

This allows you to specify the image for deployment.  For example, to use the
test version of shipyard you could deploy like:

```bash
curl -sSL https://shipyard-project.com/deploy | IMAGE=shipyard/shipyard:test bash -s
```

### PREFIX

This changes the prefix of the container names when deployed.  The default
prefix is `shipyard`.


```bash
curl -sSL https://shipyard-project.com/deploy | PREFIX=shipyard-test bash -s
```

### SHIPYARD_ARGS

These are additional controller arguments for the Shipyard controller.  See
the [Controller Usage](/docs/usage/controller/) for details.

```bash
curl -sSL https://shipyard-project.com/deploy | SHIPYARD_ARGS="--ldap-server=ldap.example.com --ldap-autocreate-users" bash -s
```

### TLS_CERT_PATH

This causes the deployment to enable TLS for all components of the system.
The proxy, swarm and controller will all be configured for TLS.  This is an
opinionated configuration and the deployer expects the following.

The certificates in the specified path must be named as follows:

- `ca.pem`: Cerfiticate Authority cert
- `server.pem`: Server certificate - this must have a SAN of `proxy` for the
internal Docker engine proxy to work properly
- `server-key.pem`: Server certificate private key
- `cert.pem`: Client certificate
- `key.pem`: Client certificate key

These will be placed in a volume container and shared amongst the various
components.  If you need to debug, you can link this container to your debug
container.  The data container name is `$PREFIX-certs`.

If you need to create certificates, [CertM](https://github.com/ehazlett/certm/)
will generate all of the needed certificates with a single command:

```bash
docker run --rm \
    -v $(pwd)/certs:/certs \
    ehazlett/certm \
    -d /certs \
    bundle \
    generate \
    -o shipyard \
    --host proxy \
    --host 127.0.0.1
```

You can then specify this path when deploying:

```bash
curl -sSL https://shipyard-project.com/deploy | TLS_CERT_PATH=$(pwd)/certs bash -s
```

## Adding a Node

The Shipyard deploy script will automatically setup a key/value store.
To add additional nodes to the Swarm cluster, you can use this script again
by specifying `node` for the `ACTION`.  For example, if the IP of your initial
node is `10.0.0.10` you can add a node by running the following:

```bash
curl -sSL https://shipyard-project.com/deploy | ACTION=node DISCOVERY=etcd://10.0.1.10:4001 bash -s
```
