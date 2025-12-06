![Database Configuration Preview](https://api.webstray.com/starlight/user/webstraycom/repository/acbs-database-configuration?customTitle=Database%20Configuration&sharpProgress=true&borderRadius=40)
## About

This repository contains information about deploying the `acbs-database` (backend and frontend) on a server.

### Built With

To deploy the `acbs-database` on a server, you will need:
- Node.js
- Express.js
- MongoDB
- Nginx

**Node.js** is the application runtime, **Express.js** is a backend web application framework for Node.js, **MongoDB** is a NoSQL database used for data storage. **Nginx** web server is used as a `reverse proxy` because, although Express.js can act as an internet-facing web server and even support SSL/TLS out of the box, it is better to use the **Nginx reverse proxy** for performance optimization, as well as for increased security, scalability, and flexibility of the application.

## Installation

First, you'll need to install `Node.js` and `MongoDB` (including **mongosh**). Then, you'll need to create the **acbs-database** directory and go into it:

```cmd
mkdir acbs-database
cd acbs-database
```

In this directory, you need to create two nested directories for the frontend and backend:

```cmd
mkdir acbs-database-backend
mkdir acbs-database-frontend
```

And then clone the acbs-database frontend and backend git repositories into these directories:

```cmd
git clone https://github.com/webstraycom/acbs-database-backend.git acbs-database-backend
git clone https://github.com/webstraycom/acbs-database-frontend.git acbs-database-frontend
```

Then go to the `acbs-database-backend` directory and install the required dependencies:

```cmd
cd acbs-database-frontend
call npm install
cd ..
```

Then go to the `acbs-database-frontend` folder and also install the required dependencies:

```cmd
cd acbs-database-backend
call npm install
cd ..
```

To run the project, use the `npx nodemon app.js` command, for example, to run the backend:

```cmd
cd acbs-database-backend
npx nodemon app.js
```

By default, the backend runs on port **3000** and the frontend on **3001**, but you can change the port to any other by changing this line in the `app.js` file:
```javascript
app.listen(3000);
```

**Congratulations!** The main part is complete, but to complete the installation, we'll need to set up a `reverse proxy`using the **Nginx** web server.

## Setting Up Reverse Proxy
  
First, we'll need to install the **Nginx** web server. Go to the [installation page](https://nginx.org/en/download.html) and download the latest stable build of Nginx for Windows.

Then unzip the downloaded Nginx archive into the `acbs-database` directory, and then create the `nginx/sites-enabled` folder. In this directory, create two files – `example.conf` and `api.example.conf`.
  
The `example.conf` file should contain the following:

```nginx
server {

	listen 80;

	server_name example.com;

	location / {

		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $remote_addr;
		proxy_pass http://localhost:3001;
	}
}
```

This configuration file is used to have the web server process requests to port **80** (default) of `example.com` and redirect them to the application running on port **3001**.
  
The `api.example.conf` file, in turn, should contain the following:

```nginx
server {

	listen 80;

	server_name api.example.com;

	location / {

		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $remote_addr;
		proxy_pass http://localhost:3000;
	}
}
```

The `api.example.conf` configuration file, in turn, is used to have the web server process requests to port **80** of the `api.example.com` subdomain and redirect them to the application running on port **3000**.

### Setting Up SSL

In order for your websites to be accessible via `https`, you need to issue **SSL certificates** for them.
The easiest way to obtain and renew certificates from **Let's Encrypt** is to use the `Certbot` tool.

To install `Certbot` and issue certificates, follow these steps:
1. **Download** `Certbot` from the [provided link](https://github.com/certbot/certbot/releases/download/v2.9.0/certbot-beta-installer-win_amd64_signed.exe) and **install** it on your system.
2. **Start** the certificate issuance process with the following command:

```cmd
certbot certonly --webroot
```

Then enter the names of the domains for which you want to issue an **SSL** certificate, separated by commas or spaces:

```cmd
example.com, api.example.com
```

Wait for the SSL certificates to be issued, and then modify the **nginx** configuration files for the domains by adding another `server` block to listen for requests on port **443**.

For example, for the domain `api.example.com`, another `server` block should look like this:

```nginx
server {

	listen 443 ssl;

	server_name api.example.com;

	ssl_certificate C:\Certbot\live\api.example.com\fullchain.pem;
	ssl_certificate_key C:\Certbot\live\api.example.com\privkey.pem;

	location / {

		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $remote_addr;
		proxy_pass http://localhost:3000;
	}
}
```

You can also set up automatic redirection from `http` to `https` by making the following changes to the first `server` block:

```nginx
server {

	listen 80;

	server_name api.example.com;

	return 301 https://$host$request_uri;
}
```

The line `return 301 https://$host$request_uri;` in the example above returns a **permanent redirect** (HTTP 301) to the `https` version of the same URL.

After all the changes have been made, your `api.example.conf` file should look like this:

```nginx
server {

	listen 80;

	server_name api.example.com;

	return 301 https://$host$request_uri;
}

server {

	listen 443 ssl;

	server_name api.example.com;

	ssl_certificate C:\Certbot\live\api.example.com\fullchain.pem; # managed by Certbot
	ssl_certificate_key C:\Certbot\live\api.example.com\privkey.pem; # managed by Certbot

	location / {

		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $remote_addr;
		proxy_pass http://localhost:3000;
	}
}
```

**Great!** You've just issued **SSL** certificates and configured your **nginx** web server to serve them. This will make your websites accessible via the secure `https` protocol.

After all the previous points have been completed, go to the `nginx/conf` folder and edit the `nginx.conf` file so that it looks like this:

```nginx
worker_processes 1;

events {

	worker_connections 1024;
}

http {

	include mime.types;
	include "/acbs-database/nginx/sites-enabled/*.conf";

	sendfile on;
	keepalive_timeout 65;
}
```

**Congratulations!** You've just successfully configured `nginx reverse proxy` for your domain and subdomain. 
All that's left for you to do is start the web server by running the following commands:

```cmd
cd acbs-database/nginx
nginx.exe
```

Your web server will now act as a `reverse proxy`, accepting requests for each domain on the port specified in its configuration file (in this case, **80** or **443**) and forwarding them to applications running on the ports also specified in the configuration files (in this case, **3000** and **3001**).
