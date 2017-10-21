##Resolve Cross Domain While Front-end Application depoly to another server

###Option 1 - Use Nginx reverse proxy 
####How to use?

Modify /etc/nginx/nginx.conf

	server {
		listen       80 default_server;
		listen       [::]:80 default_server;
		server_name  _;
		root         /usr/share/nginx/html/front-web;
		# Load configuration files for the default server block.
		include /etc/nginx/default.d/*.conf;
	
		location / {
			proxy_http_version 1.1;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection "upgrade";
			proxy_set_header Host $host;
		}
	
		location ^~ /api {
		   	proxy_pass http://www.api.com;
			proxy_redirect          off;
			proxy_set_header        X-Real-IP       $remote_addr;
			proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
		}
	}

And restart nginx.

The URL start with `/api` will redirect to `http://www.api.com`

#####Example

`Http://loction:80/api/v1/getName` -> `Http://www.api.com/api/v1/getName`

###Option 2 - Use Node.js http-proxy 

####How to use?

Use Node module express and http-proxy-middleware 

	var express = require('express');
	var proxy = require('http-proxy-middleware');
	var app = express();
	app.use('/api', proxy({target: 'http://www.api.com', changeOrigin: true}));
	app.use('/', proxy({target: 'http://localhost:80', changeOrigin: false}));
	app.listen(80);
	
### Done!!!

####The front-end can access other source api without Same-Origin Policy.