# openvpn_autoconf

A set of scripts for automatic administration of VPN
servers. Generates keys and crypto parameters for you,
installs openvpn and generates configurations for users.

## Intended audience

People already familiar with Openvpn who want to automate or
accelerate openvpn administration.
People who are starting with Openvpn administration; for
those this framework can provide a neat reference on the
basic steps required for setting up a vpn.

This is not an easy one-click setup script, because it is
by far not stable enough for that and setting up a vpn
requires still a bit of knowlege about networking and
cryptography.

It is expected that the users of this script can read the
source code and understand the concepts and modify it if
necessary.

## Usage

In general you will use the cfgtool script to administrate
the vpn; it provides commands for the major steps involved
in setting up a VPN with OpenVPN.
It needs to be run as root.

There are a couple of configuration files that are relevant:
* easy-rsa.cfg – Contains configurations for easy-rsa – the
  suite of scripts to automate key and certificate
  generation
* openvpn-client.cfg.proto, openvpn-server.cfg.proto,
  openvpn-common.cfg.proto – Configuration used by the
  OpenVPN client, server and both
* sysv-service.proto – The systemV service that will be used
  to start the VPN
* variables.cfg – All the .proto files have support for
  variables; which variables are available is defined in
  here. This contains general settings that must be edited.

### Before setting up the VPN

Decide how you want to use your VPN: What port will it
listen on? Will it use TCP or UDP? You will have to
configure your firewall to allow traffic.

If you intend to route between the VPN network and some
connected network, you will have to configure your network's
routers to route packets to the VPN network via the host
with the VPN server.

You will also need to check out openvpn_autoconf on your
computer.

```
git clone --recursive https://github.com/SoftwearDevelopment/openvpn_autoconf.git /etc/openvpn/VPN_NAME
```

(you need to replace VPN_NAME with a decent name).
Make sure, vendor/easy-rsa is also checked out. If not,
delete the clone and clone again using the --recursive flag.

Decide if you want to use your distribution's OpenVPN; it
needs to be very moder for that. (At least version 2.3.8).
You can check with `openvpn --version`.
If you don't want to use your distribution's OpenVPN, simply
uninstall it. openvpn_autoconf will download and compile
it's own automatically.

### OpenVPN Setup

1. Edit the configuration files. Look at all of them,
   specifically variables.cfg
2. Add any routes to openvpn-server.cfg.proto (look for the
   push instruction)
3. Run `cfgtool autoinstall`; this should add an init.d
   script, generate openvpn-server.cfg and openvpn-common.cfg,
   download and compile openvpn and generate keys for the
   server, the ca and dh parameters (which will take quite
   a while)
4. Start the vpn; The command for this should have been
   printed by autoinstall.
5. Run `cfgtool add_user <name>` for yourself. This should
   generate a tar file and instructions how to use it.
   Use the tar file and check if the VPN works.
6. Generate configurations for any other users.
7. Drink a beer and watch steven universe or something.

## Cryptographic considerations

### Assumptions, without whom the entire VPN would be insecure.

All of the private keys (that is CA, server and client keys)
are not password protected. Normally it would be best to use
password protected keys, unfortunately this makes automatic
startup and shutdown harder.

Client certificates are generated and stored by this script.
Normally you would generate the key and the certificate
signing request on the client's computer and never expose
it. The you would send the CSR to the CA admin and have them
sign the CSR – that is generate a real certificate from it.
This has the advantage, that an attacker could not gain
access to the vpn, even if they where eavesdropping on the
communication; remeber, the private key was never sent over
the wire.
Unfortunately, this process is very hard for people
unexperienced with crypto.

Both of the points above are major security problems; in
order for the vpn to be still secure, a couple of
assumptions must be met:
* The tar files with the client configuration are sent over
  a very secure connection; if this can not be achieved,
  consider the CSR process described above and using
  encrypted client private keys.
* The hosts in the network accessible through the network
  are protected with a VPN. If this is not possible,
  consider password protecting your clients' private keys.
  This is protection in case someone gets access to the
  client's computer.
* The CA is only used for this VPN; nothing else. If this is
  not the case, consider password protecting the CA private
  key.
* The CA resides on the VPN server. Otherwise, consider
  password protecting your CA.

The reasoning for the last two points is, that then the CA
only protects the network behind the VPN, nothing else. If
an attacker gains access to the CA, then likely got access
to your VPN server, thus also penetrating the VPN network.

### Choice of crypto algorithms and key length

In general all algorithms and key lengths where chosen for
optimal security; generally following the rule "to times the
recommended security", because I am no a security expert.

The arguments against high settings is usually a tradeoff
with performance; in practice though, performance
differences are usually unnoticeable.

You probably should not change the cyphers, hashes, key
lengths, TLS suites or activate LZO compression.  Except if
you know you know you are making the system more secure or
in very specific use cases: If you need to work with old VPN
versions (consider compiling openvpn with this script on
those machines), if you need very low latency or if you need
to transmit massive data.

### Updates

Make sure you always look for security bugs; always update
OpenVPN, this script, openssl, gnutls and openssh.

## Caveats

* The OpenVPN compiled by cfgtool will be built without
  liblzo and libpam. This is not a problem except you
  already know it is for your use case.

## TODO

* LZO Support
* Systemd support
* Better support for managing the keys externally
* Support for using encrypted (password protected) keys
