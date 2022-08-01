# Docker-Flask-NGINX-uWSGI

[blog post](https://kwangjong.github.io/2022/08/01/flask-nginx-uwsgi-docker/)

## Building a Simple Web Server
Here's how I built the server using Docker on `Ubuntu 22.04 LTS`

```shell
             ┌───────────────────────────────────────────────┐
             | server:                                       |
             |                                               |
clients <---------> nginx <---------> uwsgi <--------> flask |
           5000               5050        callable object    |
             └───────────────────────────────────────────────┘
```

### Installing and setting up Docker and Docker-compose
Install `docker`.
```shell
$ sudo apt-get install curl
$ curl -s https://get.docker.com | sudo sh
```
Add user to the `docker` group.
```shell
$ sudo usermod -aG docker $USER
```
Install `docker-compose`.
```shell
$ sudo curl -L "https://github.com/docker/compose/releases/download/v2.8.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

### File Structure

```shell
server
├── docker-compose.yml
├── flask
│ ├── Dockerfile
│ ├── app.py
│ ├── requirements.txt
│ └── uwsgi.ini
└── nginx
    ├── Dockerfile
    └── nginx.conf
```

### `./flask`
`app.py`
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "hello docker🐳"
        

if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

`requirements.txt`
```python
flask==2.1.*
click==8.0.*
itsdangerous==2.1.*
jinja2==3.0.*
markupsafe==2.1.*
werkzeug==2.2.*
uwsgi==2.0.*
```

`Dockerfile`
```python
FROM python:3

WORKDIR /app

ADD . /app
RUN pip install -r requirements.txt

CMD ["uwsgi","uwsgi.ini"]
```

`uwsgi.ini`
```python
[uwsgi]
wsgi-file = app.py
callable = app
socket = :5050
processes = 2
threads = 2
master = true
vacum = true
chmod-socket = 660
die-on-term = true
```

### `./nginx`

`nginx.conf`
```python
server {

	listen 5000;
	
	location / {
		include uwsgi_params;
		uwsgi_pass flask:5050;
	}
}
```

`Dockerfile`
```python
FROM nginx

RUN rm /etc/nginx/conf.d/default.conf

COPY nginx.conf /etc/nginx/conf.d/
```

### `docker-compose.yml`
```python
version: "3.7"

services: 
    flask:
        build: ./flask
        container_name: flask
        restart: always
        environment: 
            - APP_NAME=FlaskTest
        expose:
            - 5050

    nginx:
        build: ./nginx
        container_name: nginx
        restart: always
        ports:
            - "5000:5000"
```

## Build and start containers
```shell
$ docker-compose up -d --build
```
* `-d`: run containers in the background
* `--build`: build images before starting containers
