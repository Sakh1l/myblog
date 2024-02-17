---
title: "Docker - Build, Ship & Run"
description: "In this artcile, i will show you the complete steps on building a small web app and ship it to docker-hub image registary and run it in other system independently"
pubDate: "Feb 17 2024"
heroImage: "/blog-placeholder-3.jpg"
---

First will run a test server vm in our machine - make sure you have the cpu and ram capacity need.

We are doing this to show you the demo. You can try yourself very much on your physical system because Docker in itself a next level to VM as they are small, less resource intensive, has a quick startup and runtime and with container management tool or an orchestration tool, we can scale it high

Lets, setup the installation of Docker, we will go to [https://docs.docker.com/] to refer any doubts which is readily available.

We don’t need Docker desktop, If you are a old linux user, you already know the answer.

if you are new to linux, not alone Docker Desktop, for any Desktop Environment applications, which has a point and click actions/event called as GUI Graphical User Interface will always be resource intensive.

and if you stop and think for a second, why do we need to suffer inside a terminal (like me), instead of taking an easy approach, we are in to the paradoxical abstraction of simple is hard.

Meaning, if you think it is easy, you haven’t understood the problem. And, when you understand the problem it becomes easy.

To the topic, Linux is by nature, friendly with its approach on system design having the client server architecture adopted within its system. With that, Docker Engine is the core process/service running continuously called dockerd on the backend which we call it a deamon (a unknown force) runs until we interrupt it.

Docker has a convenience script at [https://get.docker.com/] to install it on the linux OS

run the curl -fsSL [https://get.docker.com](https://get.docker.com/) -o [get-docker.sh](http://get-docker.sh/)

sudo groupadd docker

sudo usermod -aG docker $USER

now login to docker hub using the cli. docker login

**Running Docker Server from local machine:**

when you run the command Docker version, it will show the detailed description of Docker server and client information. As i said, you can operate the server (the other remote machine) without ssh-ing into it when you set the export variable with the remote server name and ip address.

export the environment variable in the local machine

export DOCKER_HOST=ssh:///username@ipaddress

now if you run docker context ls it will show you the details of the environment variable set

you can unset the env variable with unset DOCKER_HOST

## Lets build a small Python app

```python
from flask import Flask, render_template
import subprocess

app = Flask(__name__)

def get_fortune():
   try:
       result = subprocess.check_output(['/usr/games/fortune', '-s'], stderr=subprocess.STDOUT, text=True)
       fortune = result.strip()
   except subprocess.CalledProcessError:
       fortune = "Error: 'fortune' command not available"
   return fortune

@app.route('/')
def display_fortune():
   fortune = get_fortune()
   return render_template('index.html', fortune=fortune)

if __name__ == '__main__':
   app.run(host='0.0.0.0', port=8000)
```

once, we tested the [app.py](http://app.py) in our local environment, now we can create the Dockerfile and move the source code to containerize the app

```python
# Use an official Python runtime with minimal dependencies as the parent image
FROM python:3-slim

# Install Flask and the system package for fortune
RUN pip install flask
RUN apt-get update && apt-get install -y fortune

# Create a directory for the application
WORKDIR /app

# Copy the Python script and HTML template into the container
COPY app.py /app/
COPY templates/ /app/templates/

# Expose port 8000 for the web application
EXPOSE 8000

# Set the startup command
CMD ["python", "app.py"]
```

and then once we compiled, we can build the Docker image with the below build command

```bash
docker build -t your_image_name:your_tag path/to/your/Dockerfile/directory
```

if it got compiled without any errors, you can see the built image with command

docker images

Now, we can push the build image to our Docker repository by issuing command

You should be logged-in your terminal to push or pull images from Docker Hub. Same as github this is an image repository of Docker and there are several others as well like Azure, AWS images repository or your company might have one on their own to maintain their private repository.

```bash
docker push your_repository_name/your_image_name:your_tag
```

You can pull the publicly available/hosted images and run it in your machine
