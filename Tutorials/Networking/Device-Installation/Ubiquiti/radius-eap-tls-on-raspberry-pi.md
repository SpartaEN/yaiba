# Radius with eap-tls on Raspberry pi

**NOT WORKING FOR ME, AND I HAVE NO PLAN TO RETRY**

Install radius on Raspberry pi to enable unifi WPA2 Enterprise

## Step 1

Install freeredis

```bash
sudo apt update
sudo apt install freeradius
```
## Step 2

Obtain certs

Edit `/etc/freeradius/3.0/certs/ca.cnf`, `/etc/freeradius/3.0/certs/server.cnf`, `/etc/freeradius/3.0/certs/client.cnf` and execute bootstrap to obtain certificates.

## Step 3

Configure freeradius

Edit `/etc/freeradius/3.0/clients.conf`, change `secret` to your AP to authentication password and `ipaddr` to allowed access address.

Edit `/etc/freeradius/3.0/mods-enabled/eap` as follows.

```
default_eap_type = tls
private_key_password = <password to your server.key>
private_key_file = <path to your server.key>
certificate_file = <path to your server.pem>
ca_file = <path to your ca.pem>
random_file = /dev/random
check_crl = yes
# Enhance your ciphers
cipher_list = "HIGH"
ecdh_curve = "secp384r1"
name = "EAP-TLS"
persist_dir = "${logdir}/tlscache"
```

## Step 4

Configure default site

```bash
ln -sf /etc/freeradius/3.0/sites-available/tls /etc/freeradius/3.0/sites-enabled/tls
```

Edit `/etc/freeradius/3.0/sites-enabled/default`, comment `chap` and uncomment `eap`;

Edit `/etc/freeradius/3.0/sites-enabled/tls`, change `tls` section to fit your settings.

## Step 5 (Optional)

Setup crl

Edit `/etc/freeradius/3.0/certs/crl.conf` as follows

```
[ ca ]
default_ca		= CA_default

[ CA_default ]
dir			= ./
certs			= $dir
crl_dir			= $dir/crl
database		= $dir/index.txt
new_certs_dir		= $dir
certificate		= $dir/ca.pem
serial			= $dir/serial
crl			= $dir/crl.pem
private_key		= $dir/ca.key
RANDFILE		= $dir/.rand
name_opt		= ca_default
cert_opt		= ca_default
default_days		= 60
default_crl_days	= 30
default_md		= sha256
preserve		= no
policy			= policy_match
crlDistributionPoints	= URI:http://www.example.com/example_ca.crl

[ policy_match ]
countryName		= match
stateOrProvinceName	= match
organizationName	= match
organizationalUnitName	= optional
commonName		= supplied
emailAddress		= optional

[ policy_anything ]
countryName		= optional
stateOrProvinceName	= optional
localityName		= optional
organizationName	= optional
organizationalUnitName	= optional
commonName		= supplied
emailAddress		= optional

[ req ]
prompt			= no
distinguished_name	= cacrl
default_bits		= 2048
input_password		= <password1>
output_password		= <password2>
x509_extensions		= v3_ca

[certificate_authority]
countryName		= <COUNTRY_CODE>
stateOrProvinceName	= Radius
localityName		= <REGION>
organizationName	= FreeRadius
emailAddress		= freeradius@localhost 
commonName		= "FreeRadius Certificate Authority"

[v3_ca]
subjectKeyIdentifier	= hash
authorityKeyIdentifier	= keyid:always,issuer:always
basicConstraints	= CA:true
crlDistributionPoints	= URI:http://www.example.com/example_ca.crl
```

Then execute the following commands

```bash
openssl ca -gencrl -keyfile ca.key -cert ca.pem -out crl.pem -config crl.cnf
cat ca.pem crl.pem > cacrl.pem
```