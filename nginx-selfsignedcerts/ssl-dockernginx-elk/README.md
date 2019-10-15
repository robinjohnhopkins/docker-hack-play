# Self signed certs and nginx

https://www.johnmackenzie.co.uk/post/creating-self-signed-ssl-certificates-for-docker-and-nginx/

JAN 26, 2019

## create self signed cert

```
openssl req -newkey rsa:2048 -nodes -keyout nginx/my-site.com.key -x509 -days 365 -out nginx/my-site.com.crt
Generating a 2048 bit RSA private key
.................................................+++
..............+++
writing new private key to 'nginx/my-site.com.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) []:usa
State or Province Name (full name) []:gl
Locality Name (eg, city) []:gl
Organization Name (eg, company) []:rp
Organizational Unit Name (eg, section) []:rp
Common Name (eg, fully qualified host name) []:com.my-site
Email Address []:rjh@hotmail.com
```

Once that is done you will have two new files in your nginx dir

```
- nginx/
  - my-site.com.crt
  - my-site.com.key
```

## Mounting our new key/pair into our container

docker-compose.yml

```
version: '2'
services:
  server:
    image: nginx:1.15
    volumes:
      - ./site:/usr/share/nginx/html
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/my-site.com.crt:/etc/nginx/my-site.com.crt # New Line!
      - ./nginx/my-site.com.key:/etc/nginx/my-site.com.key # New Line!
    ports:
    - "8080:80"
```

## Opening port 443 on our nginx container

docker-compose.yml

```
version: '2'
services:
  server:
    image: nginx:1.15
    volumes:
      - ./site:/usr/share/nginx/html
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/my-site.com.crt:/etc/nginx/my-site.com.crt
      - ./nginx/my-site.com.key:/etc/nginx/my-site.com.key
    ports:
    - "8080:80"
    - "443:443" # Docker open external port 443 and redirect all traffic to internal port 443
```

## Configure Nginx to serve my-site.com over https using the self signed certificate

nginx/nginx.conf

```
events {
  worker_connections  4096;  ## Default: 1024
}

http {
    server {
        listen 80;
        server_name my-site.com;
        root         /usr/share/nginx/html;
    }

    server { # This new server will watch for traffic on 443
        listen              443 ssl;
        server_name         my-site.com;
        ssl_certificate     /etc/nginx/my-site.com.crt;
        ssl_certificate_key /etc/nginx/my-site.com.key;
        root        /usr/share/nginx/html;
    }
}
```

https://my-site.com to see the results, you will get a not secure certificate warning,
but hey ho.

## nginx ssl protect kibana add elk

Taken the basic ssl nginx example that secured one page, and transposed to elk
protecting the kibana url

https://my-site.com:5601

add data via http to logstash on port 5000

```
nc localhost 5000
{"data":"a one line","name":"jane"}
{"data":"a one line","name":"time"}
{"data":"a one line","name":"tim"}
{"data":"what","name":"fred"}
```
