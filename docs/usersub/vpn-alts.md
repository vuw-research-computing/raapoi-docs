## VPN alternatives

While the official way to connect to Rāpoi is using the Cisco Anyconnect client, there are some alternatives if that is causing problems.

### ECS ssh bastions

ECS users can come in via the ECS ssh bastions ```greta-pt.ecs.vuw.ac.nz``` or ```barretts.ecs.vuw.ac.nz```.  Note that this will only work for users with valid ECS credentials.

The best way to do this is with a **ProxyJump** either in the ssh command directly, or you can add it to your ssh_config file.

Directly
```bash
ssh -J <ecs user>@barretts.ecs.vuw.ac.nz <raapoi user>@raapoi.vuw.ac.nz
```

Or via you ssh_config file

```bash
Host raapoi
    HostName raapoi.vuw.ac.nz
    ProxyJump <ECS user>@greta-pt.ecs.vuw.ac.nz
    User <Raapoi user>
```

### OpenConnect

[OpenConnect](https://www.infradead.org/openconnect/) is an opensource implimentation of the Cisco anyconnect clients.  Some users find this easier to use than the official client.

As all VPN connections currently require 2FA you need to use the [openconnect-sso python package](https://github.com/vlaci/openconnect-sso).  This has only been tested in Linux, it should be possible to make this work in a windows WSL2 terminal as well as in MacOS, but it may require modifications.

Step 0: make sure you've already setup the Microsoft Authenticator App on your phone or setup SMS as the 2FA method.

```bash
sudo apt install pipx
pipx install "openconnect-sso[full]"  # can just use "openconnect-sso" if Qt 5.x already installed
pipx ensurepath
```
Last line required because of the 'Note' that displays when you run ```pipx install "openconnect-sso[full]"```
If you have Qt 5.x installed you can skip 2 to 3 and instead: ```pipx install openconect-sso]```

Now you can connect using CLI:
```bash
openconnect-sso --server vpn.vuw.ac.nz --user <firstname.lastname>@vuw.ac.nz
```

It remembers the last used server so after the first time you can reconnect with just
```bash
openconnect-sso
```

You then just leave a tab of your command line open on this, and in a different tab connect to Raapoi.

If this doesn't work for you, ogten due to the difficulty in resolving the QT5 dependancies on macOS silicon you could try [Alternative Openconnect-sso](vpn-alts2.md)