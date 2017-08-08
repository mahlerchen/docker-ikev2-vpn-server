# IKEv2 VPN Server on Docker

[![Build Status](https://img.shields.io/docker/build/commure/docker-ikev2-vpn-server.svg)](https://hub.docker.com/r/commure/docker-ikev2-vpn-server/builds/)
[![Downloads](https://img.shields.io/docker/pulls/commure/docker-ikev2-vpn-server.svg)](https://hub.docker.com/r/commure/docker-ikev2-vpn-server/)

Recipe to build the [`commure/docker-ikev2-vpn-server`](https://hub.docker.com/r/commure/docker-ikev2-vpn-server/) Docker image (forked from [gaomd/docker-ikev2-vpn-server](https://github.com/gaomd/docker-ikev2-vpn-server)).

## Usage

This image is designed for testing with an Azure VPN Gateway, so relies on some
pretty specific network settings. Only shared secret authentication is
supported.

### 1. Create Docker Network

Create a Docker network with a specific subnet. For example:

```
docker network create azure_test --subnet=192.133.0.0/16
```

### 2. Quickstart

The container must be privileged and run with a specific IP address, for example:

```
docker run --privileged -d \
  --name vpn \
  --restart=unless-stopped \
  -p 500:500/udp \
  -p 4500:4500/udp \
  --network azure_test \
  --ip 192.133.0.254 \
  --env IPSEC_LEFT=192.133.0.254 \
  --env IPSEC_RIGHT=13.64.198.33 \
  --env IPSEC_RIGHT_SUBNET=10.3.0.0/24,10.3.200.0/29 \
  --env IPSEC_SHARED_SECRETS=": PSK abc123" \
  commure/docker-ikev2-vpn-server
```

## Configuration

### Parameters

> `IPSEC_LEFT`

IP address of the VPN server on the Docker network. This _must_ match the
value of the `--ip` parameter.

> `IPSEC_RIGHT`

Public IP address of the Azure VPN Gateway.

> `IPSEC_RIGHT_SUBNET`

Comma separated list of subnets behind the Azure VPN Gateway. Hosts
on these subnets will have VPN access.

> `IPSEC_SHARED_SECRETS`

Contents of the `ipsec.secrets` file (see below).

### Secrets

There are two ways to specify the contents of the IPSec secrets file. First,
you can mount a secrets file as a volume by adding the following to the `docker run` command:

```
-v /path/to/ipsec.secrets:/etc/ipsec.secrets
```

The second way is to specify the file contents in an environments variable
called `IPSEC_SHARED_SECRETS`. For example, by adding the following to the
`docker run` command:

```
--env IPSEC_SHARED_SECRETS=": PSK abc123"
```

This will create a shared secret with value `abc123`, available to all clients.

If you do not mount a file or specify the environment variable, a random shared
secret will be generated for you. So see its value, run:

```
docker exec -it vpn cat /etc/ipsec.secrets
```

## License

This software is licensed under the [MIT License](LICENSE).

See also [original repo](https://github.com/gaomd/docker-ikev2-vpn-server).
