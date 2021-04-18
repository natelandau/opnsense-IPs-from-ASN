# OPNsense IP Addresses from ASN

This repository aims to make creating selective routing rules targeting an entire company possible within an OPNsense firewall. OPNsense does provide support for creating HOST aliases which use FQDNs. However, these aliases do not support wildcards which makes it nearly impossible to selectively route, say, every Facebook domain to a specific firewall rule.

**Note:** Blocking wildcard domains at the DNS level is possible out-of-the-box using Unbound within OPNsense.

### Why use selective routing rules?
Some scenarios in which you may want to use this script are:

* Routing traffic to a specific company through a VPN
* Allowing outgoing or incoming traffic from a specific company or IP range

### Limitations
This script is a hammer, not a scalpel.  It will add *ALL* IP ranges from the targeted companies to the rule.  To create rules for specific URLs, add them to a `Host(s)` alias in `OPNsense --> Firewalls --> Aliases`

### How it works
This included shell script gathers the IP ranges associated with the ASNs specified in the configuration file. It then writes these IP addresses into a text file that can be read from an OPNsense `URL Table (IPs)` alias.

You can [read this document](https://www.arin.net/resources/guide/asn/) to learn more about ASNs.

# Installation and Usage
These instructions are for installing directly on a device running OPNsesnse.

1. SSH in to OPNsense
2. Install Git: `pkg install git`
3. Clone this repository:
   ```bash
   git clone https://github.com/natelandau/opnsense-IPs-from-ASN.git
   ```
4. Enter the directory: `cd opnsense-IPs-from-ASN`
5. Edit the configuration file (`SETTINGS.conf`) to reflect your preferences
6. Create a symbolic link to `/usr/local/bin/`
   ```bash
   sudo ln -s ${HOME}/opnsense-IPs-from-ASN/ips_from_asn.sh /usr/local/bin/ips_from_asn.sh
   ```
7. Run the script with sudo: `sudo ips_from_asn.sh`

## Default Configuration
The default configuration assumes you are running this script directly within OPNsense

A file named `privacy_cidrs.txt` is created containing all the IP addresses.  This file is written to a directory named `custom_aliases` will is created at OPNsense's web root (`/usr/local/www`). This can be customized with the `OUTPUT_FILE` variable

Other settings should be self-evident within `SETTINGS.conf`

### ASNs included
By default ASNs the following companies are included:

| Company               | ASN     |
| ---                   | ---     |
| AppNexus              | AS29990 |
| Bytedance (TikTok)    | AS396986 |
| Facebook              | AS32934 |
| Facebook Corp         | AS54115 |
| Facebook-offnet       | AS63293 |
| Google                | AS15169 |
| Oracle                | AS792   |
| Twitter               | AS35995 |

To add a new company's IP addresses, follow these steps.

 1. run `dig companyname.com` from a command line to get a sample IP address
 2. Copy one of the IP addresses
 3. Paste it into this tool: https://hackertarget.com/as-ip-lookup/
 4. If the ASN returned belongs to that company you're good to go. IF, on the other hand the company is a different network provider, you likely don't want to add it to this script. (ie, adding a CDN is a bad idea.)
 5. Add the ASN number to the `ASN_LIST` array in `SETTINGS.conf`

### CLI Flags
```
 -I  --IPV6        Collect IPV6 addresses.  Defaults to IPV4 only.
 -h, --help        Display help and exit
 -n, --dryrun      Non-destructive. Makes no permanent changes.
 -q, --quiet       Quiet (no output)
 -v, --verbose     Output more information. (Items echoed to 'verbose')
 ```

 ## Configuring OPNsense
 To use this file in an OPNsense firewall rule, follow these steps

 1. Log in to your OPNsense web administration area
 2. Navigate to `Firewall --> Aliases`
 3. Click the `+` button to add a new alias
 4. Name it something
 5. Select type `URL Table (IPs)`
 6. Set it to `30 days` as a `Refresh Frequency` (These IPs don't update frequently)
 7. In the `Content` field add: `https://127.0.0.1/custom_aliases/privacy_cidrs.txt`
 8. Add a `Description` if you want
 9. `Save` and `Apply` the alias
 10. Build whatever firewall rules you want

## Add the script to cron
To update these IPs, you need to add the script to cron

1. Log in to your OPNsense web administration area
2. Navigate to `System --> Administration --> Cron`
3. Click the `+` button to add a new cron task
4. Change `Day of the Month` to `1` and leave everything else as a default
5.
