# https-capture

This repo contains a small setup that aids local interception of HTTPs traffic.
The concept is old, but I figured it be helpful for me to have a place to put such a setup.

The idea is to run two proxies:

* The first one is called `reverse` in the config, and will terminate HTTPS and forward it to the second as HTTP locally.
* The second one is called `proxy` (yay, naming 👌) in the config and will accept HTTP and forward it as HTTPS to the intended `target`.

This enables us to capture the plain HTTP traffic locally.

## Preliminaries

You'll need:

* [mkcert](https://github.com/FiloSottile/mkcert)
* docker and docker-compose
* tcpdump/wireshark something like that

## Setup local http proxy to ssl target:

In `proxy/default.conf` adjust the following:

* Substitute `server www.target.tld:443;` with the target IP address you want to use
  * You can use a DNS entry, too - but that makes it necessary to pay attention to altering your local `/etc/hosts` at the right time.
* Adjust the lines for `server_name` and `proxy_set_header` to the appropriate domain.

## Setup local ssl reverse proxy:

In `reverse/default.conf` adjust the following:

* If not on MacOS pay attention to the `server host.docker.internal:27374;` line.
  * You may need to put a different host here.
* Use `mkcert www.target.tld` to generate a cert for your target
  * Use `mkcert -install` to install the certificate into your trust store
  * Put the appropriate files in `ssl_certificate` and `ssl_certificate_key` entries
* Adjust `server_name` and `proxy_set_header` as fitting

## Usage

* Start containers using `docker-compose up`
* Redirect local traffic to `reverse` container by putting `127.0.0.1 $target` in `/etc/hosts` or similar
* Capture traffic on local loopback

## Cleanup

* In `reverse` execute: `mkcert -uninstall`
* From `/etc/hosts` remove `127.0.0.1 $target`
