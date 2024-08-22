# Caddy reverse proxy setup with Crowdsec plugin

This Ansible repo sets up a Caddy reverse proxy for my homelab services. It integrates with the Crowdsec plugin that runs on my OPNsense box.

## How?

Dustin Casto wrote an outstanding guide on this setup on [his website](https://homenetworkguy.com/how-to/set-up-caddy-reverse-proxy-with-lets-encrypt-and-crowdsec-using-opnsense-lapi/). I automated it with Ansible.

Caddy runs on a VM in [Proxmox](https://www.proxmox.com/en/) on a DMZ network. I access it via [Tailscale](https://tailscale.com/), and it redirects me to the desired service.

Every service runs with a valid HTTPS certificate. I use a wildcard certificate because it's easier to manage. I also found that retrieving certificates for each subdomain is very unreliable on my network and might take hours.

I also do some SSH hardening and set up a UFW firewall.

## Prerequisites

1. Cloudflare account with an API key with permissions: `Zone.Zone Read` and `Zone.DNS Edit`. You need to set up this key `roles/reverse_proxy/files/caddy_override.conf` file
2. I use OPNsense as my firewall. However, t should be possible to modify this playbook to just install and set up Caddy.
3. A domain name. Since we want valid HTTPS certificates.
4. Ubuntu. This set-up has been tested on Ubuntu 24.04, but should work on any Debian-based system.
5. Public key for SSH authentication is set in the root of the repository in the `files/ansible.pub` file.

### Set up Crowdsec on OPNsense

This repo assumes that you have set up Crowdsec on OPNsense according to [Dustin's guide](https://homenetworkguy.com/how-to/set-up-caddy-reverse-proxy-with-lets-encrypt-and-crowdsec-using-opnsense-lapi/). During the playbook's run, it will pause and ask you to validate the Caddy machine in OPNsense. Make sure you do it as described in the guide.

### Set up Cloudflare

This repo builds Caddy with the Cloudflare plugin to perform the DNS-01 challenge to validate that you own a domain. [This guide](https://homenetworkguy.com/how-to/replace-opnsense-web-ui-self-signed-certificate-with-lets-encrypt/) might be helpful

## Variables and files

Define your host variables in `host_vars/caddyDMZ.yml`. The comments in the example file should be helpful.

Set up your [Caddyfile](https://caddyserver.com/docs/caddyfile).

Set up `caddy_override.conf`. It only contais Cloudflare API token. Keep it **safe**!

Remove the `.example` suffix from the provided files.

## How to run?

1. Provided you have Ansible installed and `inventory` and `ansible.cfg` files created, you need to run `ansible-playbook bootsrap.yml`. It will create a specified Ansible user and give it `sudo` rights. From now on, you can run any Ansible playbook as this user.
2. Run `ansible-playbook caddy.yml`.

## Things to keep in mind

1. I use `unattended-upgrades` to upgrade the system. It also _might_ reboot if it's necessary after the update. That's not ideal if you want 100% availability and stability because things can break after an update. Ubuntu is a rather stable system, and I only automatically install security updates; it _should_ not break after a reboot.
2. I took steps to harden SSH and set up a firewall, but it is not a super-protected system. I do not expose any ports on my home network and use public key authentication for SSH, so this security level works for me.
3. If you are inside your home network, you can set up OPNsense to override IP address that are returned by DNS. Now, if you request `service.example.com`, you will not query a remote DNS server, and the domain will resolve immediately to your local IP. See [Unbound DNS Override Aliases in OPNsense](https://homenetworkguy.com/how-to/create-unbound-dns-override-aliases-in-opnsense/) for more information.

## What's missing?

This is still work-in-progress. I use [Tailscale](https://tailscale.com) to access my network. Currently, I install and set up Tailscale manually.

## 🙌 Thanks

1. Dustin Casto for the amazing guide, [Set Up a Caddy Reverse Proxy with Let's Encrypt and CrowdSec Using OPNsense LAPI](https://homenetworkguy.com/how-to/set-up-caddy-reverse-proxy-with-lets-encrypt-and-crowdsec-using-opnsense-lapi/), this repo is based.
2. Jay LaCroix's excellent [Ansible video series](https://www.youtube.com/playlist?list=PLqyUgadpThTL1guZCdGy7H8V4snPrpj8t).
3. Jim's Garage for [changing the way I create new VMs](https://youtu.be/Kv6-_--y5CM).
4. Software developers whose work made this repo possible.
