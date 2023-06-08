# WebSockets and Video Streaming
To enable any sort of recording whether it's video or audio streaming, you will need a domain and an SSL certificate, so follow the steps in the `Domains_and_SSL.md` file first.

Once you have your domain and SSL and you can access your instance through `https://yourdomain.com` come back here.

## Nginx Configuration
Here is a quick example of an infrastructure that includes:
* A static page: The `index.html` file is in `/home/ubuntu/camera-streamer/website/`
* Another page with webcam access through javascript: This can be accessed via `https://yourdomain.com/server/`
* A different page that receives the video feed through javascript: This can be accessed via `https://yourdomain.com/client/` and redirects to whatebver is on port `5000`
* WebSocket based communications on port `3006`, it uses the same domain but different path `../ws/..`
* Code-Server accessible through `https://yourdomain.com/code/` and redirects to port `3800` so also accessible via `https://yourdomain.com:3800`

### WebApp configurations
```nginx
server {
	listen 80 default_server;
	listen [::]:80 default_server;
	root /home/ubuntu/camera-streamer/website/;

	# Add index.php to the list if you are using PHP
	index index.html index.htm index.nginx-debian.html;

	server_name yourdomain.com www.yourdomain.com;

    location / {
  	  try_files $uri /index.html;
	    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	    proxy_set_header Host $host;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection upgrade;
      proxy_set_header Accept-Encoding gzip;
    }

	location /client/ {
		alias /home/ubuntu/camera-streamer/clients/;
		index index.html index.htm index.nginx-debian.html;
		try_files $uri $uri/ /client/index.html;
	}

	location ~^/server/(.*)$ {
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header Host $host;
		proxy_pass http://127.0.0.1:5000/$1;
		proxy_set_header Host $host;		
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection upgrade;
		proxy_set_header Accept-Encoding gzip;
	}

	 location ~^/ws/(.*)$ {
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header Host $host;
		proxy_pass http://127.0.0.1:3006/$1;
		proxy_set_header Host $host;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection upgrade;
		proxy_set_header Accept-Encoding gzip;
  }

	location ~^/code/(.*)$ {
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header Host $host;
		proxy_pass http://127.0.0.1:3800/$1;
    proxy_set_header Host $host;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection upgrade;
		proxy_set_header Accept-Encoding gzip;
	}

  listen [::]:443 ssl ipv6only=on; # managed by Certbot
  listen 443 ssl; # managed by Certbot
  ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem; # managed by Certbot
  include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
```

### Websockets and Socketio
If your app needs to communicate via websockets or socketio, you can add rules like this to your nginx configuration.

For instance, here we are enabling websocket and socketio communication on ports `3000` and `3001`.I'm also adding some examples of access to websockets via subdomains.
```nginx
upstream websocket {
	server 127.0.0.1:3000;
	server yourdomain.com:3000;
	server dev.yourdomain.com:3000;
	server test.yourdomain.com:3000;
	server localhost:3000;

	server 127.0.0.1:3001;
	server yourdomain.com:3001;
	server dev.yourdomain.com:3001;
	server test.yourdomain.com:3001;
	server localhost:3001;
  ...
}

upstream socketio_nodes {
  ip_hash;

  server 127.0.0.1:3000;
  server yourdomain.com:3000;
  server dev.yourdomain.com:3000;
  server test.yourdomain.com:3000;
  server localhost:3000;

  server 127.0.0.1:3001;
  server yourdomain.com:3001;
  server dev.yourdomain.com:3001;
  server test.yourdomain.com:3001;
  server localhost:3001;
  ...
}
```

### Code-Server config
Here is a quick snippet of what a `code-server` configuration might look like. In this way, you can offer your webapp through `https://yourdomain.com` and access your code-server instance through `https://code.yourdomain.com` or `https://yourdomain.com:3800`.

```nginx
server {
  listen 80;

  server_name code.yourdomain.com www.code.yourdomain.com;

  location / {
    proxy_pass http://127.0.0.1:3800/;
    proxy_set_header Host $host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection upgrade;
    proxy_set_header Accept-Encoding gzip;
  }

  listen [::]:443 ssl; # managed by Certbot
  listen 443 ssl; # managed by Certbot
  ssl_certificate /etc/letsencrypt/live/code.yourdomain.com/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/code.yourdomain.com/privkey.pem; # managed by Certbot
  include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

```

You can also check the official `code-server` guide here: https://github.com/coder/code-server/blob/main/docs/guide.md#using-a-subdomain