# WaybackProxy

WaybackProxy is a retro-friendly HTTP proxy which retrieves pages from the [Internet Archive Wayback Machine](http://web.archive.org) or [OoCities](http://www.oocities.org) and delivers them in their original form, without toolbars, scripts and other extraneous content that may confuse retro browsers.

![1999 Google viewed on Internet Explorer 4.0 on Windows 95](http://i.imgur.com/tXsLc6O.png)

## Setup

Python 3.5 or newer is required.

1. Edit `config.json` to your liking (make sure to set `QUICK_IMAGES` to `false` for non-Internet-connected systems)
2. Optionally exclude domains from being proxied by adding them to `whitelist.txt`
3. Install dependencies: `pip install --user -r requirements.txt`
4. Start `waybackproxy.py`
5. Set up your retro browser:
	* If your browser supports proxy auto-configuration (PAC), set the auto-configuration URL to `http://ip:port/proxy.pac` where `ip` is the IP of the system running WaybackProxy and `port` is the proxy's port (8888 by default).
	* If proxy auto-configuration is not supported or fails to work, set the browser to use an HTTP proxy at that IP and port instead.
	* Transparent proxying is also supported for advanced users, with no configuration to WaybackProxy itself required.
		* The easiest way to set up a transparent WaybackProxy is to run it on port 80 ([this cannot be done on Linux without security implications](https://unix.stackexchange.com/questions/87348/capabilities-for-a-script-on-linux)\), set up a fake DNS server - such as `dnsmasq -A "/#/ip"` where `ip` is the IP of the system running WaybackProxy - to redirect all requests to the proxy, and point client machines at that DNS server.
6. Try it out! You can edit most settings that are in `config.json` by browsing to http://web.archive.org while on the proxy, although you must edit `config.json` to make them permanent.
7. Press Ctrl+C to stop the proxy

## Docker Container

A Dockerfile is included that allows you to run WaybackProxy from a Docker container. 

### Environment Variables

When deploying via Docker, the config.json can be customized by specifying environment variables when creating the docker container. The environment variables match the example config.json in this repository. Below is a complete list:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `HOST` | (blank) | Host address to bind to for the HTTP proxy (leave blank to bind on all interfaces). Only useful when running the container with `host` networking. |
| `LISTEN_PORT` | `8888` | Listen port for the HTTP proxy. Only useful when running the container with `host` networking. |
| `DATE` | `20011025` | Date to get pages from Wayback. YYYYMMDD, YYYYMM and YYYY formats are accepted, the more specific the better. |
| `DATE_TOLERANCE` | `365` | Allow the client to load pages and assets up to X days after DATE. Set to `null` to disable this restriction. |
| `GEOCITIES_FIX` | `true` | Send Geocities requests to oocities.org if set to `true`. |
| `QUICK_IMAGES` | `true` | Use the original Wayback Machine URL as a shortcut when loading images. The browser must have an Internet connection that can reach the Wayback Machine for this to work. |
| `WAYBACK_API` | `true` | Use the Wayback Machine Availability API to find the closest available snapshot to the desired date, instead of directly requesting that date. |
| `CONTENT_TYPE_ENCODING` | `true` | Allow the Content-Type header to contain an encoding. |
| `SILENT` | `true` | Disables logging to STDOUT if set to `true`. |
| `SETTINGS_PAGE` | `true` | Enables the settings page on http://web.archive.org if set to `true`. |

### How to run in Docker

#### Using Docker Registry

To pull:

```bash
docker pull richardg867/waybackproxy:latest
```
To run:

```bash
docker run -d -e DATE=20011025 -p 8888:8888 richardg867/waybackproxy
```

#### Build locally

To build:

```bash
docker build --no-cache -f Dockerfile -t waybackproxy .
```
To run:

```bash
docker run -d -e DATE=20011025 -p 8888:8888 waybackproxy
```

## Known issues and limitations

* The Wayback Machine itself is not 100% reliable. Known issues include:
  * Pages newer than the specified date (setting a specific YYYYMMDD date instead of a wider YYYYMM or YYYY helps with that);
  * Random broken images;
  * Strange 404 errors caused by bad server responses or incorrect URL capitalization at archival time;
  * Infinite redirect loops;
  * Server errors when it's having a bad day.
* The Wayback Machine blocks IPs for a few seconds (with *Connection refused* errors) if connections are made too quickly. WaybackProxy has measures to try and prevent this block, but it may still be triggered if:
	* `QUICK_IMAGES` is used alongside the PAC file, as that bypasses WaybackProxy and connects to the Wayback Machine directly;
  * Multiple instances of WaybackProxy on the same Internet connection are being used at once.
* WaybackProxy will work around some redirection scripts (example: `http://example.com/redirect?to=http://...`) which are not archived by the Wayback Machine, but the destination URLs are sometimes not archived either.
* WaybackProxy is not a generic proxy. HTTP POST, CONNECT and other methods are only supported on excluded domains for passthrough purposes.
* Transparent proxying mode requires HTTP/1.1 and therefore cannot be used with some really old (pre-1996) browsers. Use standard mode with such browsers.

## Other links

* [**Donate to the Internet Archive**](https://archive.org/donate/), they need your help to keep the Wayback Machine and its petabytes upon petabytes of data available to everyone for free with no ads.
* [Check out 86Box](https://86box.net), the emulator I use for testing WaybackProxy on older browsers.
* [WaybackProxy container](https://hub.docker.com/r/richardg867/waybackproxy) on Docker Hub.
