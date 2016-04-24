# Contributing to DPlatform

#### This document is attended to provide informations for people who wants to add their contribution to the DPlatform project

## Shell Style Guide

DPlatform is made in a POSIX Shell script. Therefore, your contribution must comply to this following coding standards:
- POSIX compliance (`#!/bin/sh`): all commands must work with sh, dash, bash, ksh, zsh
- Unless required no semicolons
- Indentation whenever is possible (for example except for `cat <<EOF EOF`)
- Use a space and a semicolon before `then` and `do`: `if/elif [ ] ;then`, and `while/for ;do`
- If you have simple conditions, use one-line booleans expressions like `[ = ] && [ ] || command` instead of `if/elif/else`

## Application installation script structure

Each application installation scripts are built upon a same structure. Here we suppose that you application is called ***MyApp***

First at the begining af each application script file, there are command to update and remove itself.
```sh
if [ $1 = update ] ;then
  cd /home/myapp/MyApp
  git pull
  whiptail --msgbox "MyApp updated!" 8 32
  break
fi
[ $1 = remove ] && sh sysutils/services.sh remove MyApp && userdel -r myapp && whiptail --msgbox "MyApp removed!" 8 32 && break
```
Depending of your app, you can ask to the user to write some arguments, like a port number.
``` sh
# Define port
port=$(whiptail --title "MyApp port" --inputbox "Set a port number for MyApp" 8 48 "80" 3>&1 1>&2 2>&3)
```

Next, the prerequisites
```sh
# Prerequisites of MyApp that have builtin support in DPlatform's sysutils, for example NodeJS or MongoDB
. sysutils/NodeJS.sh
. sysutils/MongoDB.sh

# Add MyApp user
useradd -m myapp

# Go to myapp user directory
cd /home/myapp

# Install MyApp dependencies, for example python, gcc...
$install python gcc
```

Follows the install instructions, that depends of your app
```sh
# If your app has a git repository
git clone https://github.com/MyApp/MyApp

# Download the arcive
wget https://myapp.com/myapp_version1.tgz -O myapp.tgz 2>&1 | \
stdbuf -o0 awk '/[.] +[0-9][0-9]?[0-9]?%/ { print substr($0,63,3) }' | whiptail --gauge "Downloading the archive..." 6 64 0

# Extract the downloaded archive and remove it
(pv -n myapp.tgz | tar xzf -) 2>&1 | whiptail --gauge "Extracting the files from the archive..." 6 64 0
rm myapp.tgz

cd MyApp

#****************************#
# Some installation commands #
#****************************#

# Change the owner from root to myapp
chown -R myapp /home/myapp
```

Create a SystemD service for your app
```sh
# Create the SystemD service
cat > "/etc/systemd/system/etherdraw.service" <<EOF
[Unit]
Description=MyApp Server
After=network.target
[Service]
Type=simple
WorkingDirectory=/home/myapp/MyApp
ExecStart=/usr/bin/node server.js
Environment=ROOT_URL=http://$IP:$port/ PORT=$port
User=myapp
Restart=always
[Install]
WantedBy=multi-user.target
EOF

# Start the service and enable it to start on boot
systemctl start myapp
systemctl enable myapp
```
You can also use the builtin SystemD service creation of DPlatform
```sh
# Add SystemD process and run the server
sh sysutils/services.sh MyApp "/usr/bin/node /usr/bin/MyApp" /home/myapp/MyApp myapp
# This command correspond to
# sh ServiceCreationScriptPath Description= "ExecStart=" WorkingDirectory= User=

```
Finally, print a message box to inform that the installation is finished and the URL access of your app
```sh
whiptail --msgbox "MyApp installed!

Open http://$URL:$port in your browser." 10 64
```