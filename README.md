# Xahau RPC/WSS Submission Node 
Xahau submission node installation with nginx &amp; lets encrypt TLS certificate.

---

This script will take the standard Xahau node install (non-docker version) and supplement it with the necessary configuration to provide a TLS secured RPC/WSS endpoint using Nginx.

This script is automating the manual steps in here, please review for more information about the process: https://github.com/go140point6/xahl-info/blob/main/setup-xahaud-node.md 

# Table of Contents
---

- [Xahau RPC/WSS Submission Node](#xahau-rpcwss-submission-node)
- [Table of Contents](#table-of-contents)
  - [Current functionality](#current-functionality)
  - [How to download \& use](#how-to-download--use)
    - [How to UPDATE from older scripts like go140point6](#how-to-update-from-older-scripts-like-go140point6)
    - [Clone the repo And install](#clone-the-repo-and-install)
    - [Config files](#vars-file-for-advanced-users)
    - [Nginx related](#nginx-related)
    - [Permitting Access](#node-ip-permissions)
    - [Testing your Xahaud server](##testing-your-xahaud-server)
      - [xahaud](#xahaud)
      - [RPC and API](#rpc-and-api)
      - [Websocket](#websocket)
       - [locally](#locally)
       - [remotley](#remotely-safe-to-use-on-your-evernode)
  - [Future updates](#future-updates)
    - [Contributors:](#contributors)
    - [Feedback](#feedback)


---
---

## Current functionality
 - Install options for Mainnet (Testnet if demand warrants)
 - Supports the use of custom variables using the `xahl_node.vars` file
 - Saves all data from questions to .env file, so it can be used to check/prompt if setup run again
 - Detects UFW firewall & applies necessary firewall updates.
 - Installs & configures Nginx 
   - sets up nginx so that it splits the incoming traffic to your supplied domain 5 ways
        - 1.static website for allowed IPs
        - 2.static webbsite for blocked IPs 
        - 3.the main node websocket(wss)
        - 4.any rpc traffic (apis from POST)
        - 5.public access to .toml file
   - TL;DR; you only need ONE domain pointing to this server.
   - Automatically detects the IPs of your ssh session, the node itself, and its local enviroment then adds them to the nginx_allowlist.conf file
 - Applies NIST security best practices
 
---

# How to download & use

To download the script(s) to your local node & install, read over the following sections and when ready simply copy and paste the code snippets to your terminal window.

## How to UPDATE from older scripts like go140point6

older versions, where the allow list was saved in `/etc/nginx/sites-available/xahau` WILL need saving FIRST, (as we now have a unified allowlist file)

        sudo nano /etc/nginx/sites-available/xahau

to open them into nano, or you what maybe useful as you can scroll easier is to

        sudo cat /etc/nginx/sites-available/xahau

now you can either write down your ip allow list manually, or copy and paste them into another notepad, where you can copy and paste them back into the auto generated nginx_allowlist.conf when install has finished.

## Clone the repo and Install

whatever folder you git clone to, is the place it will use to clone the xahaud image, and where the nginx allowlist will be, so initially it would be good to;

        cd ~

which changes working directory into your "home" directory of the user you are in.

        apt install git
        git clone https://github.com/gadget78/xahl-node
        cd xahl-node
        chmod +x *.sh

now install with;

        sudo ./setup.sh

this this go through a serious of questions (in blue) and output all info along the way

one of the questions for example is to eneter additional IPs into the allow list, 

when finished, it will give a little nfo, on how to check its working,

also shows you where the new `nginx_allowlist.conf` file is. just in case you need to enter more in future (doing command `nginx -s reload` after the edit)


### Vars file for Advanced users

there is a xahl.node.vars file, to adjust how the setup.sh is configured, but this is only needed for advanced users

to adjust the default settings via this file, edit it using your preferred editor such as nano;

        nano ~/xahl-node/xahl_node.vars

there are MANY things that can be adjusted, but the main useful ones are

- `ALWAYS_ASK` - "true" so setup can be configured to not ask you if there is an entery already, useful to re-generate files/setting
- `INSTALL_UPDATES` - "true" this setting can be used to turn off the checking/installing up linux updates, and default install packages
- `VARVAL_CHAIN_NAME` - "mainnet" this is the main "mode" of setup
- `INSTALL_UFW` - "true" chose to install or not
- `INSTALL_CERTBOT_SSL` - "true" VERY useful for switch off the need for SSL to be installed, so it can be install behind another front end
- `INSTALL_LANDINGPAGE` - "true" so you can switch off the landing page generation
- `INSTALL_TOML` - "true" so you can switch of .toml file generation

#### .env file

all the questions asked in setup, are saved in file called `.env` this is so they dont get altered by updateding the repo (git pull)

- `USER_DOMAIN` - your server domain.
- `CERT_EMAIL` - email address for certificate renewals etc.
- `TOML_EMAIL` - email address for the PUBBLIC .toml file.
- `XAHAU_NODE_SIZE` - allows you to state the "size" of the node, this will change the amount of RAM, and HDD thats used.

---

# Nginx related

All the domain specific config is contained in the file `NGX_CONF_NEW`/xahau (this and `default` is deleted, and recreated when running the script)

logs are held at /var/log/nginx/

and Although this works best as a dedicated host with no other nginx/proxy instances,

it can work behind another instance, you may need to adjust the setting in the main nginx.conf file to suit your enviroment, mainly so the enabled the allowlist to work correctly.

for example, in etc/nginx/nginx.conf you may need to adjust/add these lines under `http`

        `set_real_ip_from 172.16.0.0/12; # with the IP set to you exsisting proxy IP`
        `real_ip_header X-Real-IP;`
        `real_ip_recursive on;`

# Node IP Permissions

the setup script adds 3 by default to the nginx_allowlist file, the detected SSH IP, the external nodes IP, and the local enviroment IP.

In order to add/remove access to your node, you adjust the addresses within the `nginx_allowlist.conf` file

edit the `nginx_allowlist.conf` file with your preferred editor e.g. `nano nginx_allowlist.conf`.

start every line with allow, a space, and then the IP, and end the line with a semicolon.

for example

        allow 127.0.0.0;
        allow 192.168.0.1;

__ADD__ : Simply add a new line with the same syntax as above,

__REMOVE__ : Simply delete the line.

THEN

__RELOAD__ : for the changes to take effect you need to issue command `sudo nginx -s reload`

---

# Testing your Xahaud server

This can be done simply by entering the domain/URL into a browser.

this will give you either a notice that you IP is blocked, and which IP to put in the access list.

or if not blocked, will use the RPC function of your node, and pull all the basic details

or following these next examples, you can test it manually...

### XAHAUD

this option ONLY works LOCALLY on the xahau node, and is directly quering the node itself (type in the terminal;

        xahaud server_info

Note: look for `"server_state" : "full",` and your xahaud should be working as expected.
it may be in a `connected` state, if just installed. Give it time.

### RPC and API

a simple command from terminal, which can be locally on the xah node, or externally on a different terminal, a great way to test connection from your Evernode to your Xahau node

    curl -X POST -H "Content-Type: application/json" -d '{"method":"server_info"}' https://your.domain

if it works, will reply with your server info, if not, will reply with a raw html file (of your blocked page for example)

### WEBSOCKET

#### LOCALLY

to test the Websocket function of your xahau node, we can use the wscat command, 
BUT this is not safe to install on a already exsisting/running evernode. only locally, or non-evernode terminal

Copy the following command replacing `yourdomain.com` with your domain you used in setup. (this can be found in the .env file)

        wscat -c wss://yourdomain.com

This should open another session within your terminal, similar to the below;

    Connected (press CTRL+C to quit)
    >

and enter

    { "command": "server_info" }
which will return you xah node server info

#### REMOTELY safe to use on your evernode

another tool to test websocket function is via node, first we check/install websocket function

        node instal ws

now the command to perform, may be best to copy and paste this, then alter the wss://your.domain to the domain to test

        node -e "const ws = new (require('ws'))('wss://nota.network'); ws.once('open', () => { console.log('WebSocket Working'); ws.close(); }).on('error', () => console.log('WebSocket Failed'));"

which will reply WebSocket working, or Websocket Failed

---

# Future Updates

as all your "saved" data from questions are in .env file, we can simple

apply repo updates to your local clone, 

        cd ~/xahl-node

which changes your working directory to the directry where you cloned repo last time

        git pull

checks for new repo updates, and pulls new updates if there is any.

if you HAVE chnaged the .vars file, you wll need to perferm a "stash" that override those setting.. 

        git stash

then you can then perform git pull, and get the latest updates again .. 


---

### Contributors:  
This was all made possible by [@inv4fee2020](https://github.com/inv4fee2020/), this is 98% his work, I just copied pasta'd... and fixed his spelling mistakes like "utilising"... ;)

A special thanks & shout out to the following community members for their input & testing;
- [@nixer89](https://github.com/nixer89) helped with the websocket splitting
- [@realgo140point6](https://github.com/go140point6)
- [@gadget78](https://github.com/gadget78)
- [@s4njk4n](https://github.com/s4njk4n)
- @samsam

---

### Feedback
Please provide feedback on any issues encountered or indeed functionality by utilizing the relevant Github issues..