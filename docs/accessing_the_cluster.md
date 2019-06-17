# Accessing the Cluster

To access Rāpoi, you'll first need to get an account provisioned for you by contacting the [CAD research support team](../support) with your VUW staff username.
If you don't have a VUW staff account, it may still be possible to be given access - please [contact us](../support) to determine options.

_Access is via SSH_

*  Hostname: raapoi.vuw.ac.nz
*  IP Address: 130.195.19.14
*  Port: 22
*  Username: Your VUW username
*  Password: Your VUW password

*NOTE:* On-campus wired network connection or [VPN](https://vpn.vuw.ac.nz/+CSCOE+/logon.html#form_title_text).  VPN is required if
connecting from campus wifi or from off-campus. Some users have had issues with
using the hostname and instead need to use the IP address, eg
`harrelwe@130.195.19.14`

More information on VUW VPN services can be found [here](https://www.victoria.ac.nz/its/staff-services/core-tools-and-services/remote-access).


### SSH Clients
_Mac OSX SSH Clients_
You can use the built-in Terminal.app or you can download iTerm2 or XQuartz. XQuartz is required to be installed if you wish to forward GUI applications (matlab, rstudio, xstata, sas, etc), aka X forwarding.

* Terminal.app is the default application for command-line interface
  * To login using the built-in Terminal.app on Mac, go to
    * Applications --> Utilities --> Terminal.app
    * Or use Spotlight search (aka Command-Space)
* [iTerm2](https://www.iterm2.com/) is a good replacement for the default Terminal app
* [XQuartz](https://www.xquartz.org/) is a Xforwarding application with its own terminal.  XQuartz can be used in conjuction with the Terminal.app for GUI apps.  NOTE: Mac users should run the following command: `sudo defaults write org.macosforge.xquartz.X11 enable_iglx -bool true`   We have found that this allows some older GUI applications to run with fewer errors.
 

NOTE:  Once at the command prompt you can type the following to login (replace "username" with your VUW user):

`ssh username@raapoi.vuw.ac.nz`

_Windows SSH Clients_

* Recommended Clients:
  * [Git Bash](https://gitforwindows.org/) is a great option and is part of the Git for Windows project
  * [MobaXterm](https://mobaxterm.mobatek.net/) is a good option, especially if you require access to GUI applications such as MATLAB or xStata.  This also has a built-in SFTP transfer window.
