apt install snapd\
snap install --classic certbot\
ln -s /snap/bin/certbot /usr/bin/certbot

##################################

vi /lib/systemd/system/energizet-box.service

	[Unit]
	Description=Energizet Box service

	[Service]
	Type=simple
	WorkingDirectory=/var/box
	ExecStart=/var/box/Energizet.Box.Web --urls=http://localhost:5001

	[Install]
	WantedBy=multi-user.target

systemctl start energizet-box.service\
systemctl enable energizet-box.service

vi /etc/nginx/conf.d/energizet-box.conf

	server {
			server_name     box.energizet.ru;
			location / {
					proxy_pass      http://localhost:5001/;
			}
	}

certbot --nginx

	server {
			server_name     box.energizet.ru;
			location / {
					proxy_pass      http://localhost:5001/;
			}

		listen 443 ssl; # managed by Certbot
		ssl_certificate /etc/letsencrypt/live/box.energizet.ru/fullchain.pem; # managed by Certbot
		ssl_certificate_key /etc/letsencrypt/live/box.energizet.ru/privkey.pem; # managed by Certbot
		include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
		ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

	}
	server {
		if ($host = box.energizet.ru) {
			return 301 https://$host$request_uri;
		} # managed by Certbot


			listen  80;
			server_name     box.energizet.ru;
		return 404; # managed by Certbot


	}
	

###########################################################

vi /lib/systemd/system/energizet-history.service

	[Unit]
	Description=Energizet History service

	[Service]
	Type=simple
	WorkingDirectory=/var/history
	ExecStart=/var/history/Histoty.Storublioner.Web --urls=http://localhost:5002

	[Install]
	WantedBy=multi-user.target

systemctl start energizet-history.service\
systemctl enable energizet-history.service

vi /etc/nginx/conf.d/energizet-history.conf

	server {
			server_name     history.energizet.ru;
			location / {
					proxy_pass      http://localhost:5002/;
			}
	}

certbot --nginx

	server {
			server_name     history.energizet.ru;
			location / {
					proxy_pass      http://localhost:5002/;
			}


		listen 443 ssl; # managed by Certbot
		ssl_certificate /etc/letsencrypt/live/history.energizet.ru/fullchain.pem; # managed by Certbot
		ssl_certificate_key /etc/letsencrypt/live/history.energizet.ru/privkey.pem; # managed by Certbot
		include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
		ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

	}
	server {
		if ($host = history.energizet.ru) {
			return 301 https://$host$request_uri;
		} # managed by Certbot


			server_name     history.energizet.ru;
		listen 80;
		return 404; # managed by Certbot


	}