# letsencrypt.sh

This is a client for signing certificates with an ACME-server (currently only provided by letsencrypt) implemented as a relatively simple bash-script.

It uses the `openssl` utility for everything related to actually handling keys and certificates, so you need to have that installed.

Other dependencies are (for now): curl, sed

Perl no longer is a dependency.
The only remaining perl code in this repository is the script you can use to convert your existing letsencrypt-keyfile into something openssl (and this script) can read.

Current features:
- Signing of a list of domains
- Renewal if a certificate is about to expire
- Certificate revocation

Please keep in mind that this software and even the acme-protocol are relatively young and may still have some unresolved issues.
Feel free to report any issues you find with this script or contribute by submitting a pullrequest.

## Usage:

```text
Usage: ./letsencrypt.sh [-h] [command [argument]] [parameter [argument]] [parameter [argument]] ...

Default command: help

Commands:
 --cron (-c)                      Sign/renew non-existant/changed/expiring certificates.
 --revoke (-r) path/to/cert.pem   Revoke specified certificate
 --help (-h)                      Show help text
 --env (-e)                       Output configuration variables for use in other scripts

Parameters:
 --domain (-d) domain.tld         Use specified domain name instead of domains.txt, use multiple times for certificate with SAN names
 --force (-x)                     force renew of certificate even if it is longer valid than value in RENEW_DAYS
 --config (-f) path/to/config.sh  Use specified config file
 --privkey (-p) path/to/key.pem   Use specified private key instead of account key (useful for revocation)
```

### domains.txt

The file `domains.txt` should have the following format:

```text
example.com www.example.com
example.net www.example.net wiki.example.net
```

This states that there should be two certificates `example.com` and `example.net`,
with the other domains in the corresponding line being their alternative names.

### example nginx config

If you want to use nginx you can set up a location block to serve your challenge responses:

```
location /.well-known/acme-challenge {
  root /var/www/letsencrypt;
}
```

For this to work i'd suggest either configuring `/var/www/letsencrypt` as WELLKNOWN directory,
or to create a symlink to the default location next to the script: `ln -s /var/www/letsencrypt .acme-challenges`

## Import

### import-account.pl

This perl-script can be used to import the account key from the original letsencrypt client.

You should copy `private_key.json` to the same directory as the script.
The json-file can be found in a subdirectory of `/etc/letsencrypt/accounts/acme-v01.api.letsencrypt.org/directory`.

Usage: `./import-account.pl`

### import-certs.sh

This script can be used to import private keys and certificates created by the original letsencrypt client.

By default it expects the certificates to be found under `/etc/letsencrypt`, which is the default output directory of the original client.
You can change the path by setting LETSENCRYPT in your config file: ```LETSENCRYPT="/etc/letsencrypt"```.

Usage: `./import-certs.sh`
