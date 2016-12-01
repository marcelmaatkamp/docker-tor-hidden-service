# docker-tor-hidden-service with client authorization.

Create a tor hidden service with a link with client authorization enabled (see _HiddenServiceAuthorizeClient_ in the tor manual).

## Setup

Create a `docker-compose.yml` with the services, in this case a `hello` service and a `hidden` service which makes hello a hidden service:
```yaml
version: '2'
services:

 hello:
  image: "tutum/hello-world"
  hostname: "hello"

 tor:
# image: goldy/tor-hidden-service
  build: "docker-tor-hidden-service/"
  links:
   - "hello"
  environment:
   HELLO_PORTS: "80:80"
   HELLO_AUTH: "0123456789"
  volumes:
   - "tor:/var/lib/tor/hidden_service"

volumes:
 tor: 
```

The key to add to your `torrc` file in the tor browser can be extracted from the logfile when the hidden service starts:
```
tor_1    | Entrypoint INFO     hello: uifjb4bmt2ilpa2v.onion T3GgNOwr5ML5s5FGZsJ/CR # client: 0123456789:80
tor_1    | Dec 01 14:36:51.682 [notice] Tor 0.2.9.5-alpha (git-330846ac087f7b32) running on Linux with Libevent 2.0.22-stable, OpenSSL 1.0.2j and Zlib 1.2.8.
tor_1    | Dec 01 14:36:51.682 [notice] Tor can't help you if you use it wrong! Learn how to be safe at https://www.torproject.org/download/download#warning
tor_1    | Dec 01 14:36:51.682 [notice] This version is not a stable Tor release. Expect more bugs than usual.
tor_1    | Dec 01 14:36:51.682 [notice] Read configuration file "/etc/tor/torrc".
tor_1    | Dec 01 14:36:51.000 [notice] Parsing GEOIP IPv4 file /usr/local/share/tor/geoip.
tor_1    | Dec 01 14:36:51.000 [notice] Parsing GEOIP IPv6 file /usr/local/share/tor/geoip6.
tor_1    | Dec 01 14:36:51.000 [notice] Bootstrapped 0%: Starting
tor_1    | Dec 01 14:36:52.000 [notice] Bootstrapped 80%: Connecting to the Tor network
tor_1    | Dec 01 14:36:52.000 [notice] Bootstrapped 85%: Finishing handshake with first hop
tor_1    | Dec 01 14:36:53.000 [notice] Bootstrapped 90%: Establishing a Tor circuit
tor_1    | Dec 01 14:36:53.000 [notice] Tor has successfully opened a circuit. Looks like client functionality is working.
tor_1    | Dec 01 14:36:53.000 [notice] Bootstrapped 100%: Done
```

Add it to the `torrc` file like this:
```
HidServAuth uifjb4bmt2ilpa2v.onion T3GgNOwr5ML5s5FGZsJ/CR hello
```

### Tools

A command line tool `onions` is available in container to get `.onion` url when container is running.

```sh
# Get services
$ docker exec -ti torhiddenproxy_tor_1 onions
hello: vegm3d7q64gutl75.onion:80
world: b2sflntvdne63amj.onion:80

# Get json
$ docker exec -ti torhiddenproxy_tor_1 onions --json
{"hello": ["b2sflntvdne63amj.onion:80"], "world": ["vegm3d7q64gutl75.onion:80"]}
```

### pyentrypoint

This container is using [`pyentrypoint`](https://github.com/cmehay/pyentrypoint) to generate its setup.

If you need to use the legacy version, please checkout the `legacy` branch or pull `goldy/tor-hidden-service:legacy`.
