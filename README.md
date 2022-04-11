# Deploy flask app with nginx using gunicorn and supervisor

This is a walkthrough that illustrates how to deploy a Flask application using an easy technique.

## Installation of packages

Use the package manager apt (for Ubuntu) to install packages.

```bash
sudo apt update -y
sudo apt install python3-virtualenv -y
sudo apt install python3-pip -y
sudo apt install nginx -y
sudo apt install supervisor -y
```

## Create the working directory of your app


```bash
mkdir flask-prd
mkdir flask-prd/project1
mkdir flask-prd/project2
cd flask-prd
```

## Create Virtualenv


```bash
virtualenv env
source env/bin/activate
```

## Create the flask app inside project1 and project2


```bash
vi app.py
```
```python
from flask import Flask
app = Flask(__name__)
	
@app.route('/')
def index():
	return "Hello World from first app"
	
if __name__ == '__main__':
   app.run(debug=True)

```

```bash
vi app2.py
```
```python
from flask import Flask
app = Flask(__name__)
	
@app.route('/')
def index():
	return "Hello World from second app"
	
if __name__ == '__main__':
   app.run(debug=True)

```
## Test the App

```bash
python app.py
python app2.py
```

# Install Gunicorn

```bash
pip install gunicorn
```
## Usage

```bash
gunicorn --bind 0.0.0.0:5000 app:app
gunicorn --bind 0.0.0.0:8000 app2:app
```

# Supervisor
Supervisor will look after the Gunicorn process and make sure that they are restarted if anything goes wrong, or to ensure the processes are started at boot time.

```bash
sudo vi /etc/supervisor/conf.d/hello.conf
```
```
[program:flask-prd]
directory=/home/ubuntu/flask-prd/project1
command=/home/ubuntu/flask-prd/env/bin/gunicorn app:app -b 0.0.0.0:5000
autostart=true
autorestart=true
stderr_logfile=/var/log/hello_world/hello_world.err.log
stdout_logfile=/var/log/hello_world/hello_world.out.log
[program:flask-prd-app2]
directory=/home/ubuntu/flask-prd/project2
command=/home/ubuntu/flask-prd/env/bin/gunicorn app2:app -b 0.0.0.0:8000
autostart=true
autorestart=true
stderr_logfile=/var/log/hello_world/hello_world_app2.err.log
stdout_logfile=/var/log/hello_world/hello_world_app2.out.log
```

### To enable the configuration

```
$ sudo supervisorctl reread
$ sudo service supervisor restart
```

# Setup Nginx

Remove the default nginx configuration file and create new one

```bash
sudo rm /etc/nginx/sites-enabled/default
sudo vi /etc/nginx/conf.d/virtual.conf
```
```
server {
    listen       80;
    server_name  YOUR_PUBLIC_DNS;

    location / {
        proxy_pass http://0.0.0.0:5000;
    }
}
server {
    listen       8080;
    server_name  YOUR_PUBLIC_DNS;

    location / {
        proxy_pass http://0.0.0.0:8000;
    }
}

```
Now test the Nginx and restart it.

```bash
sudo service nginx restart
sudo nginx -t
```
