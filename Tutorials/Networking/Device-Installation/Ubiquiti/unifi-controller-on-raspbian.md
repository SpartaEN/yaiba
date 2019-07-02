# Unifi Controller on Raspbian

Install unifi controller on raspbian(also applies to linux distro with apt)

## Requirement

Raspberry pi with raspbian installed

## Step 1

Add new source to apt-get and add gpg key

```bash
echo 'deb http://www.ui.com/downloads/unifi/debian stable ubiquiti' | sudo tee /etc/apt/sources.list.d/100-ubnt-unifi.list
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv 06E85760C0A52C50 
```

## Step 2

Install unifi controller

```bash
sudo apt update
sudo apt install unifi
```

## Step 3

Start unifi controller and make it auto-start

```bash
sudo systemctl start unifi
sudo systemctl enable unifi
```

## Step 4 (Optional)

Configure unifi controller with let's encrypt

Due to the network environment, I'll take dns-01 challenge with cloudflare as an example. You can find the document of other dns-01 plugins [HERE](https://certbot.eff.org/docs/using.html#dns-plugins).

### Preparation

```bash
sudo apt update
sudo apt install python3 python3-pip
sudo pip3 install certbot certbot-dns-cloudflare
sudo mkdir /root/.secrets
sudo mkdir /root/.scripts
```

Create `cloudflare.ini` in `sudo mkdir /root/.secrets` with following contents

```ini
dns_cloudflare_email = "The email address of your cloudflare account."
dns_cloudflare_api_key = "The API key to your cloudflare account."
```

Create `post-unifi.sh` in `sudo mkdir /root/.scripts` with following contents

```bash
#/bin/sh
/usr/bin/openssl pkcs12 -export -in /etc/letsencrypt/live/<your domain>/fullchain.pem -inkey /etc/letsencrypt/live/<your domain>/privkey.pem -out /etc/letsencrypt/live/<your domain>/cert_and_key.p12 -name tomcat -CAfile /etc/letsencrypt/live/<your domain>/chain.pem -caname root -password pass:<a strong password>;
rm -f /etc/letsencrypt/live/<your domain>/keystore;
/usr/bin/keytool -importkeystore -srcstorepass <repeat the strong password> -deststorepass aircontrolenterprise -destkeypass aircontrolenterprise -srckeystore /etc/letsencrypt/live/<your domain>/cert_and_key.p12 -srcstoretype PKCS12 -alias tomcat -keystore /etc/letsencrypt/live/<your domain>/keystore;
/usr/bin/keytool -import -trustcacerts -alias unifi -deststorepass aircontrolenterprise -file /etc/letsencrypt/live/<your domain>/chain.pem -noprompt -keystore /etc/letsencrypt/live/<your domain>/keystore;mv /var/lib/unifi/keystore /var/lib/unifi/keystore-`date -I`;
cp /etc/letsencrypt/live/<your domain>/keystore /var/lib/unifi/keystore;
systemctl restart unifi;
exit 0;
```

### Install the cert

```bash
/usr/local/bin/certbot certonly --deploy-hook /root/.scripts/post-unifi.sh --dns-cloudflare --dns-cloudflare-credentials /root/.secrets/cloudflare.ini -d <your domain> --preferred-challenges dns-01
```

### Setup auto-renew

Execute `sudo crontab -e` and add the following contents

```
14 5 * * * /usr/local/bin/certbot certonly --deploy-hook /root/scripts/post-unifi.sh --dns-cloudflare --dns-cloudflare-credentials /root/.secrets/cloudflare.ini -d <your domain> --preferred-challenges dns-01
```