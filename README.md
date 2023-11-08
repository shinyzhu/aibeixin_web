# Website for aibeixin.cn

This is the code and CI/CD for website [aibeixin.cn](https://aibeixin.cn) which is my wife's bakery shop in Songjiang, Shanghai, China.

## Code

This web is built based on the [Vanilla Template](https://templatemo.com/tm-526-vanilla). I love the parallax scrolling style. Please use a Mac/PC to visit it.

The code is located under the `www` folder.

## CI

TODO:

## CD

I've set up GitHub Actions to automatically deploy the `www` to the host.

Credits to: <https://zellwk.com/blog/github-actions-deploy/>

### Deploy with Docker

I first used this approach to deploy the web into a [Tencent Cloud Lighthouse](https://www.tencentcloud.com/products/lighthouse) host which comes with Docker.

Step 1: Create the `Dockerfile`:

``` 
FROM nginx:alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY certs/ /etc/nginx/certs/
COPY www/ /var/www/
```

Step 2: Create `nginx.conf`:

```
server {
    listen 80;
    server_name aibeixin.cn;

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name aibeixin.cn;

    ssl_certificate /etc/nginx/certs/aibeixin.cn_bundle.pem;
    ssl_certificate_key /etc/nginx/certs/aibeixin.cn.key;

    root /var/www;
    index index.html;

    location / {
        #proxy_pass http://localhost:9001;
        #proxy_set_header Host $host;
        #proxy_set_header X-Real-IP $remote_addr;
        
        try_files $uri $uri/ =404;
    }
}
```

Step 3: Prepare SSL certificates, I put them under the `certs` folder.

// just skipping this

Step 4: Build the image

Send all files into the server then run:

```sh
root@lighthouse:/var/web/aibeixin#docker build . -t aibeixin_web
```

Step 5: Make it run

```sh
root@lighthouse:/var/web/aibeixin#docker run -d -p 80:80 -p 443:443 --name aibeixin_web aibeixin_web
```

It just works.

### Deploy with Nginx

I just found I need to deploy mutiple web in the only one server. So I need the SNI of Nginx.

Then I installed Nginx directly into the host.

Step 1: Install Nginx

I just followed the tutorial from [DigitalOcean here](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-22-04).

Step 2: Create `aibeixin.conf` under `/etc/nginx/sites-available/`

```
server {
    listen 80;
    server_name aibeixin.cn;

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name aibeixin.cn;

    ssl_certificate /var/www/aibeixin/certs/aibeixin.cn_bundle.pem;
    ssl_certificate_key /var/www/aibeixin/web/certs/aibeixin.cn.key;

    root /var/www/aibeixin;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Step 3: Create soft link of site configuration

```
ln -s /etc/nginx/sites-available/aibeixin.conf /etc/nginx/sites-enabled/aibeixin.conf
```

Step 4: Reload and restart Nginx

```sh
nginx -t
ystemctl restart nginx
```

## Enjoy

 😉 