# OPNsense IP Addresses from ASN

This repository aims to make creating selective routing rules targeting an entire company possible within an OPNsense firewall. OPNsense does provide support for creating HOST aliases which use FQDNs. However, these aliases do not support wildcards which makes it nearly impossible to selectively route, say, every Facebook domain to a specific firewall rule.

**Note:** Blocking wildcard domains at the DNS level is possible out-of-the-box using Unbound within OPNsense.

### Why use selective routing rules?
Some scenarios in which you may want to use this script are:

* Routing traffic to a specific company through a VPN
* Allowing outgoing or incoming traffic from a specific company or IP range

### Limitations
This script is a hammer, not a scalpel.  It will add *ALL* IP ranges from the targeted companies to the rule.  This means that
* Adding Google will include YouTube and anything hosted in Google Cloud.
* Adding Amazon (not included by default) will include Amazon Video, anything hosted in AWS, etc.

To block specific URLs, add them to a `Host(s)` alias in `OPNsense --> Firewalls --> Aliases`.

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
5. Ensure the script is executable: `chmod 750 gather_ips.sh`
6. Edit the configuration file (`SETTINGS.conf`) to reflect your preferences
7. Run the script with sudo: `sudo ./gather_ips.sh`

## Default Configuration


---
${bold}$(basename "$0") [OPTION]...${reset}

  Pulls IP ranges for common services and adds them to a text file.  This file
  can be added to a 'URL Table' alias in OPNsense to allow you to build firewall
  rules.

  IP ranges from the following companies are included.  Keep in mind, it will
  include multiple brands (i.e - Facebook includes Instagram & WhatsApp.
  Google includes YouTube and all of Azure, etc.)

  Companies:
    * Twitter           * Bytedance (Tik Tok)
    * Facebook          * Google
    * Oracle

  NOTE: Amazon is not here on purpose. Their ASN includes prime video which disallows common VPNs.

  To use this script, change the location of the OUTPUT_FILE variable and customize
  the list of company ASNs within the ASN_LIST array.

  To add a new company's IP addresses, follow these steps.

    1. run 'dig companyname.com' from a command line to get a sample IP address
    2. Copy one of the IP addresses
    3. Paste it into this tool: https://hackertarget.com/as-ip-lookup/
    4. If the ASN returned belongs to that company you're good to go. IF, on the other
       hand the company is a different network provider, you likely don't want to add
       it to this scriot
    5. Add the ASN number to the ASN_LIST array

  The README at the root of the git repository contains instructions for installing this script.

  ${bold}Options:${reset}
    -I  --IPV6        Collect IPV6 addresses.  Defaults to IPV4 only.
    -h, --help        Display this help and exit
    -l, --loglevel    One of: FATAL, ERROR, WARN, INFO, DEBUG, ALL, OFF  (Default is 'ERROR')

      $ $(basename "$0") --loglevel 'WARN'

    -n, --dryrun      Non-destructive. Makes no permanent changes.
    -q, --quiet       Quiet (no output)
    -v, --verbose     Output more information. (Items echoed to 'verbose')
    --force           Skip all user interaction.  Implied 'Yes' to all actions.