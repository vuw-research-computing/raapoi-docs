## Alternative Openconnect-sso

This is an alternative way to use the Openconnect SSO if you're having trouble getting the QT5 dependancies to resolve, which is a particular problem on macOS with M1/M2 silicon.  At some point the more official openconnect-sso will change to QT6 and this shouldn't be needed anymore.

Note for this you will be running a fork of the openconnect-sso on my github, so it's good practise to havea look at make sure I'm not doing anything nefariouss.  I'm not, but then I would say that.  Also the code is not mine, it's a fork of a pull request on another repo that I;ve setup to be the main branch to simplify this process.  Use with caution.

You'll also need a new version of chrome or chromium-browser to do this as we'll be using selenium to drive a webbrowser (in this case a chromelike browser) to handle the web part of the 2fa.

create a directory somewhere convenient, like vuwvpn or omething and cd there.
then
```bash
python3 -m venv env
source env/bin/activate  # activate our python venv
```
Then download the requirements file from my fork of the openconnect-sso.  It's not my code, I just made a repo from a pull request that hasn't been merged in yet, but we need to increase the timeout.
```bash
wget https://raw.githubusercontent.com/andre-geldenhuis/openconnect-sso/master/requirements.txt 
```

Then install those requirements, note they will install some stuff from my fork
```bash
pip install -r requirements.txt
```
Vuw's ssl stuff on the VPN 2fa uses an old ssl config so we need to downgrade the ssl on newer linux versions, we can do that just for the vpn, best not do this systemwide.
```bash
wget https://raw.githubusercontent.com/andre-geldenhuis/openconnect-sso/master/vuwssl.conf
```
Now to test as a one liner
```bash
OPENSSL_CONF=vuwssl.conf openconnect-sso --server vpn.victoria.ac.nz --user andre.geldenhuis@vuw.ac.nz 
```
If that works, ctrl-c to end it and make a script to make this easy and avoid needing to activate a python venv everytime:
Somewhere sensible to you make a file that contains the following, you'll need to adjust the paths to match yours
```bash
#!/bin/bash

source ~/vuwvpn/env/bin/activate
OPENSSL_CONF=~/vuwvpn/ssl.conf openconnect-sso --server vpn.victoria.ac.nz --user <firstname>.<lastname>@vuw.ac.nz 
```
chmod it to be executable and just run it in a separeate terminal window when you need to connect to the vpn
