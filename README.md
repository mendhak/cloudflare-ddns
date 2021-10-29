
<p align="center"><a href="https://timknowsbest.com/free-dynamic-dns" target="_blank" rel="noopener noreferrer"><img width="1024" src="feature-graphic.jpg" alt="Cloudflare DDNS"/></a></p>

# 🚀 Cloudflare DDNS

Access your home network remotely via a custom domain name without a static IP!

A small, 🕵️ privacy centric, and ⚡ lightning fast multi-architecture Docker image for self hosting projects.

## 🇺🇸 Origin

This script was written for the Raspberry Pi platform to enable low cost self hosting to promote a more decentralized internet.  
This repo was forked from [timothymiller/cloudflare-ddns](https://github.com/timothymiller/cloudflare-ddns) and some pending pull requests have been merged.  For details see the [changelog](CHANGELOG.md).  

## 🧹 More than just DDNS

`cloudflare-ddns` handles the busy work for you, so deploying web apps is less of a clickfest. Every 5 minutes, the script fetches public IPv4 and IPv6 addresses and then creates/updates DNS records for each subdomain in Cloudflare. Stale, duplicate DNS records are removed for housekeeping.

## 📊 Stats

| Size  | Downloads | Discord |
| ------------- | ------------- | ------------- |
| [![cloudflare-ddns docker image size](https://img.shields.io/docker/image-size/mendhak/cloudflare-ddns?style=flat-square)](https://hub.docker.com/r/mendhak/cloudflare-ddns "cloudflare-ddns docker image size")  | [![Total DockerHub pulls](https://img.shields.io/docker/pulls/mendhak/cloudflare-ddns?style=flat-square)](https://hub.docker.com/r/mendhak/cloudflare-ddns "Total DockerHub pulls")  

## ⁉️ How Private & Secure?

1. Uses zero-log external IPv4 & IPv6 provider ([cdn-cgi/trace](https://www.cloudflare.com/cdn-cgi/trace))
2. Alpine Linux base image
3. HTTPS only via Python Software Foundation requests module
4. Docker runtime
5. Open source for open audits
6. Regular updates

## 🚦 Getting Started

First copy the example configuration file into the real one.

```bash
cp config-example.json config.json
```

Edit `config.json` and replace the values with your own.

### 🔑 Authentication methods

You can choose to use either the newer API tokens, or the traditional API keys

To generate a new API tokens, go to your [Cloudflare Profile](https://dash.cloudflare.com/profile/api-tokens) and create a token capable of **Edit DNS**. Then replace the value in

```json
"authentication":
  "api_token": "Your cloudflare API token, including the capability of **Edit DNS**"
```

Alternatively, you can use the traditional API keys by setting appropriate values for:

```json
"authentication":
  "api_key":
    "api_key": "Your cloudflare API Key",
    "account_email": "The email address you use to sign in to cloudflare",
```

### Enable or disable IPv4 or IPv6

Some ISP provided modems only allow port forwarding over IPv4 or IPv6. In this case, you would want to disable any interface not accessible via port forward.

```json
"a": true,
"aaaa": true
```

### Other values explained

```json
"zone_id": "The ID of the zone that will get the records. From your dashboard click into the zone. Under the overview tab, scroll down and the zone ID is listed in the right rail",
"subdomains": "Array of subdomains you want to update the A & where applicable, AAAA records. IMPORTANT! Only write subdomain name. Do not include the base domain name. (e.g. foo or an empty string to update the base domain)",
"proxied": false (defaults to false. Make it true if you want CDN/SSL benefits from cloudflare. This usually disables SSH)
```

## 📠 Hosting multiple subdomains on the same IP?

You can save yourself some trouble when hosting multiple domains pointing to the same IP address (in the case of Traefik) by defining one A & AAAA record  'ddns.example.com' pointing to the IP of the server that will be updated by this DDNS script. For each subdomain, create a CNAME record pointing to 'ddns.example.com'. Now you don't have to manually modify the script config every time you add a new subdomain to your site!

## 🌐 Hosting multiple domains (zones) on the same IP?

You can handle ddns for multiple domains (cloudflare zones) using the same docker container by separating your configs inside ```config.json``` like below:

```bash
{
  "cloudflare": [
    {
      "authentication": {
          "api_token": "api_token_here", 
          "api_key": {
              "api_key": "api_key_here",
              "account_email": "your_email_here"
          }
      },
      "zone_id": "your_zone_id_here",
      "subdomains": [
        "",
        "remove_or_replace_with_your_subdomain"
      ],
      "proxied": true
    },
    {
      "authentication": {
          "api_token": "api_token_here", 
          "api_key": {
              "api_key": "api_key_here",
              "account_email": "your_email_here"
          }
      },
      "zone_id": "your_zone_id_here",
      "subdomains": [
        "",
        "remove_or_replace_with_your_subdomain"
      ],
      "proxied": true
    }
  ]
}
```

## 🐳 Deploy with Docker Compose

Pre-compiled images are available via [the official docker container on DockerHub](https://hub.docker.com/r/mendhak/cloudflare-ddns).

Modify the host file path of config.json inside the volumes section of docker-compose.yml.

```yml
version: "3.7"
services:
  cloudflare-ddns:
    image: mendhak/cloudflare-ddns:latest
    container_name: cloudflare-ddns
    security_opt:
      - no-new-privileges:true
    network_mode: "host"
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - /YOUR/PATH/HERE/config.json:/config.json
    restart: unless-stopped
```

### ⚠️ IPv6

Docker requires network_mode be set to host in order to access the IPv6 public address.

### 🏃‍♂️ Running

From the project root directory

```bash
docker-compose up -d
```

## 🐧 Deploy with Linux + Cron

### 🏃 Running (all distros)

This script requires Python 3.5+, which comes preinstalled on the latest version of Raspbian. Download/clone this repo and give permission to the project's bash script by running `chmod +x ./start-sync.sh`. Now you can execute `./start-sync.sh`, which will set up a virtualenv, pull in any dependencies, and fire the script.

1. Upload the cloudflare-ddns folder to your home directory /home/your_username_here/

2. Run the following code in terminal

```bash
crontab -e
```

3. Add the following lines to sync your DNS records every 15 minutes

```bash
*/15 * * * * /home/your_username_here/cloudflare-ddns/start-sync.sh
```

## Building from source

Create a config.json file with your production credentials.

### 💖 Please Note

The optional `docker-build-all.sh` script requires Docker experimental support to be enabled.

Docker Hub has experimental support for multi-architecture builds. Their official blog post specifies easy instructions for building with [Mac and Windows versions of Docker Desktop](https://docs.docker.com/docker-for-mac/multi-arch/).

1. Choose build platform

- Multi-architecture (experimental)  `docker-build-all.sh`

- Linux/amd64 by default  `docker-build.sh`

2. Give your bash script permission to execute.

```bash
sudo chmod +x ./docker-build.sh
```

```bash
sudo chmod +x ./docker-build-all.sh
```

3. At project root, run the `docker-build.sh` script.

Recommended for local development

```bash
./docker-build.sh
```

Recommended for production

```bash
./docker-build-all.sh
```

### Run the locally compiled version

```bash
docker run -d mendhak/cloudflare-ddns:latest
```

## License

This Template is licensed under the GNU General Public License, version 3 (GPLv3).

