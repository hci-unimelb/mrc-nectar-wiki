# Domain and SSL

## Setup a domain
You can get a domain from [Google Domains](https://domains.google.com/registrar/?pli=1) or [GoDaddy](https://www.godaddy.com/en-au).

Once you get your domain you can edit the DNS configuration so that your domain will point to your instance IP Address as such:

### Google Domains
`DNSSEC` **Enabled**

| Host name                    | Type  | TTL    | Data           |
| ---------------------------- | ----- | ------ | -------------- |
| domain.com                  | A     | 1 hour | \<ip-address\> |
| \*.code.domain.com          | A     | 1 hour | \<ip-address\> |
| code.domain.com             | A     | 1 hour | \<ip-address\> |
| www.domain.com              | CNAME | 1 hour | @.             |
| www.code.domain.com         | A     | 1 hour | \<ip-address\> |


| Name Servers |
| ------------|
| ns-cloud-e1.googledomains.com |
| ns-cloud-e2.googledomains.com |
| ns-cloud-e3.googledomains.com |
| ns-cloud-e4.googledomains.com |

## Setup SSL and HTTPS
This step is especially important if your webpage needs to use the user's webcam, record anything or even just access the clipboard.



### Using Let's Encrypt with NGINX
From: https://github.com/coder/code-server/blob/main/docs/guide.md#using-lets-encrypt-with-nginx and https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04

This option requires that the remote machine be exposed to the internet. Make sure that your instance allows HTTP/HTTPS traffic.

You'll need a domain name, follow the steps in the previous subsection. In short: you can purchase a domain from [Google Domains](https://domains.google.com) and then add an A record to your domain that contains your instance's IP address.

**(Optional) Install NGINX**
I'm assuming you already have nginx installed if you followed the steps to setup your MRC instance. But anyway, you can install it this way:
  ```bash
  $ sudo apt update
  $ sudo apt install -y nginx
  ```

**Install `certbot`**
  ```bash
  $ sudo apt-get update
  $ sudo apt-get install -y certbot python3-certbot-nginx
  ```

**NGINX config update**

Make sure to update your domain in nginx with (use whatever config file you're using now)
```bash
$ sudo nano /etc/nginx/sites-available/default.conf
```

**Get an SSL Certificate**
```bash
$ sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```
Follow the instructions and make sure you actually input correct and existing information, SSL is all about security and trust after all...

**Update other NGINX configs**
Make sure to update your nginx configurations to use your new SSL certified domain and use HTTPS instead of HTTP.

For instance here is what to change for `code-server`
Update `/etc/nginx/sites-available/code-server.conf` using sudo:

```bash
$ sudo nano /etc/nginx/sites-available/code-server.conf
```
Then edit the configuration like this:
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name yourdomain.com;

    location / {
      proxy_pass http://localhost:8080/;
      proxy_set_header Host $host;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection upgrade;
      proxy_set_header Accept-Encoding gzip;
    }
}
```
Be sure to replace `yourdomain.com` with your domain name!

At this point, you should be able to access code-server via
`https://yourdomain.com`
