---
cover: ../.gitbook/assets/main-week1.png
coverY: 92.672
layout:
  cover:
    visible: true
    size: hero
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Week 1 Dockerise App

## 0. **Learning Materials**

* Video: [FREE AWS Cloud Project Bootcamp - Week 1 — App Containerization](https://www.youtube.com/watch?v=zJnNe5Nv4tE\&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv\&index=19\&ab\_channel=ExamPro)
* Andrew's repo:&#x20;
* My branch repo: [`01-00-app-containerisation`](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/tree/01-00-app-containerisation)&#x20;

### Task List

* [ ] Containerise backend (write `Dockerfile`)
* [ ] Build & run the container&#x20;
* [ ] Containerise frontend (write `Dockerfile`)
* [ ] Build & run the container&#x20;
* [ ] Write docker-compose file for container orchestration

***

## 1.1 Workflow - Backend

We create a `Dockerfile` in the folder `backend-flask`.

{% code title="backend-flask/Dockerfile" lineNumbers="true" %}
```docker
FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY requirements.txt requirements.txt

RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```
{% endcode %}

* **line 1**: pull the `puthon slim-buster image version 3.10` from the Docker Hub.
* **line 3**: set the working directory inside the container
* **line 5**: copy the file `requirements.txt` inside the container's working directory.&#x20;
* **line 7**: run the command `pip3 install -r requirements.txt`. This will install all the dependencies listed in the requirements file.&#x20;
* **line 9**: copy everything from the root folder (where Dockerfile is located on the host machine) into the working directory inside the container.&#x20;
* **line 11**: set an environmental variable called `FLASK_ENV` inside the container.&#x20;
  * In a Flask application, `FLASK_ENV` indivates the environment in which the Flask app is running. This variable accepts there values `development`, `production` and `testing`. When `FLASK_ENV=development`, Flask providers features like debug mode, auto-reloading on code changes, and detailed error messages.
* **line 13**: expose the Docker container's port to the host machine.&#x20;
* **line 14**: the command to run flask inside the container.&#x20;
  * `--host=0.0.0.0` is our local machine (your desktop, laptop, etc.)
  * `--port` is the port for our flask server.

### **Inside or Outside the container?**

<table><thead><tr><th width="332">Outside Container</th><th>Inside Container</th></tr></thead><tbody><tr><td><code>FROM python:3.10-slim-buster</code></td><td></td></tr><tr><td></td><td><code>WORKDIR /backend-flask</code></td></tr><tr><td><code>COPY requirements.txt</code></td><td><code>requirements.txt</code></td></tr><tr><td></td><td><code>RUN pip3 install -r requirements.txt</code></td></tr><tr><td><code>COPY .</code></td><td> <code>.</code></td></tr><tr><td></td><td><code>ENV FLASK_ENV=development</code></td></tr><tr><td></td><td><code>EXPOSE ${PORT}</code></td></tr><tr><td></td><td><code>CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]</code></td></tr></tbody></table>



### What happens inside the container?

If you wonder how the docker container will be furnished (in other words, how the "image will be built"), we can visualise what will happen inside the Docker container by running the commands in the `Dockerfile` in our local machine's terminal.

* **line 7**: cd into `backend-flask`, then install `requirements.txt` on terminal.
  * `pip install -r requirements.txt`

<div data-full-width="true">

<figure><img src="../.gitbook/assets/스크린샷 2023-10-31 213143.png" alt=""><figcaption><p>Make sure you are inside the backend-flask folder.</p></figcaption></figure>

</div>

* Line 14: run a flask server.
  * `python3 -m flaks run --host=0.0.0.0 --port=4567`
  * Then go visit the backend's home endpoint (`/api/activities/home`)

<div data-full-width="true">

<figure><img src="../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

</div>

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption><p>The endpoint is inaccessible.</p></figcaption></figure>

The endpoint is inaccessible because the variables are missing. We need to set the env variables.

```python
from flask import Flask
from flask import request
from flask_cors import CORS, cross_origin
...

app = Flask(__name__)
frontend = os.getenv('FRONTEND_URL')   # these URL values have to be configured.
backend = os.getenv('BACKEND_URL')
origins = [frontend, backend]
```

Set env variables:&#x20;

* `export FRONTEND_URL="*"`
* `export BACKEND_URL="*"`

Then run the backend-flask container again. Hit the endpoint again.&#x20;

* Make sure to unlock the port on the port tab (in Gitpod workspace)
* Open the link for `4567` in your browser
* Append `/api/activities/home` to URL
  * `4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}/api/activities/home`

<figure><img src="../.gitbook/assets/image (51).png" alt=""><figcaption><p>The endpoint returns the hard-coded data.</p></figcaption></figure>

### Build a Docker image

We completed our Docker container simulation. Now we will build a Docker image for the backend. The above simulation will actually happen inside the container once we run a container based on this image.&#x20;

* Run: `docker build -t  backend-flask ./backend-flask`

<div data-full-width="true">

<figure><img src="../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

</div>

These are various Docker commands that runs a `backend-flask` container from the image we built (the image ID is `12a857404d45` in the above image for example).

* `docker run --rm -p 4567:4567 -it backend-flask`
  * `--rm`: remove the container when exiting the container.&#x20;
    * [More on this tag later in the Discussion section](./#more-on-rm).
  * `-p`: short for `--port`. Maps the host port to the container port.
  * `-it`: short for "interactive". When the container starts running, it opens the bash terminal inside the container so you can interact with your container through bash commands.
    * This is useful for checking if env variables are correctly set inside the container.
  * `backend-flask` at the end: this is the name of the Docker image. The image (name or ID) has to come always at the end of the command.
* `FRONTEND_URL="*" BACKEND_URL="*" docker run --rm -p 4567:4567 -it backend-flask`: this sets the env variables for this single run of this container. You can use this when your `Dockerfile` doesn't have env variables set.&#x20;
  * `docker run --rm -p 4567:4567 -e FRONTEND_URL='*' -e BACKEND_URL='*' -it backend-flask`
    * `-e`: the tag for environment variable. You can provide its value directly in the command.&#x20;
  * `docker run --rm -p 4567:4567 -e FRONTEND_URL -e BACKEND_URL -it backend-flask`: you can just assign env variables in case you have these variables' values set in your local machine.&#x20;
    * `set FRONTEND_URL="*"`
    * `set BACKEND_URL="*"`

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Troubleshooting Docker containers

#### Check the logs

In Gitpods, there are two ways to open view the logs. The below image shows both ways. First, in the Docker tab on the far-left column, menu-click on the container, then on the "View Logs" option. Otherwise, you can run the following command:&#x20;

* `docker logs --tail 1000 -f <CONTAINER_ID>`
  * `--tail`:  will show at least 1000 lines. Change the value for a different amount of lines.&#x20;
  * `-f`: stands for "follow". With this tag, the command will keep the log stream open, continuously showing new log entries as they occur in real-time.

<div data-full-width="true">

<figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption><p>The backend-container erroring out.</p></figcaption></figure>

</div>

Maybe something is wrong with the env variables. Let's check if the values are correctly set. Run the following command:&#x20;

* `docker exec -it <CONTAINER_ID>`
  * `exec`:  execute
  * `-it`: interactive mode

<div data-full-width="true">

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption><p>Inside the interactive docker.</p></figcaption></figure>

</div>

In the above image, the result of the logging command (`env | grep URL`) shows that the env variables are correctly set. So what's the problem?&#x20;

Well, it turns out, I didn't append the desired endpoint (`/api/activities/home`) correctly. Appending the endpoint returns the correct result.&#x20;

<div data-full-width="true">

<figure><img src="../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption><p>Now it works!</p></figcaption></figure>

</div>

Now that the backend container is running and the endpoints are responding correctly, we conclude building the docker image for the backend.

## 1.2 Workflow - Frontend

In our local machine, cd into the frontend folder, install `npm` and create a `Dockerfile`.

```bash
cd frontend-react-js
npm i
code Dockerfile
```

The `Dockerfile` code for the frontend is as below:&#x20;

{% code lineNumbers="true" %}
```docker
FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install

EXPOSE ${PORT}
CMD [ "npm", "start" ]
```
{% endcode %}

* **line 1**: pull `node of version 16.18` from the Docker hub.
* **line 3**: set env variable. Our react app for the frontend will use the port number 3000.
* **line 5**: we are currently in the `frontend-react-js` folder. This is the root directory for the frontend `Dockerfile`. Copy everything in this root folder into the frontend Docker container.&#x20;
* **line 6**: set the working directory inside the container as `/frontend-reactp-js`.
* **line 7**: install `npm` inside the container.
* **line 10**: now it's all set in the container. Run `npm start` inside the container.&#x20;

Now that the `Dockerfile` is ready, build it into an image.

* `docker build -t frontend-react-js ./frontend-react-js`

Once the image is built, we double-check it by listing the images.&#x20;

* `docker images`

{% code fullWidth="true" %}
```
gitpod /workspace/aws-bootcamp-cruddur-2023-again (01-app-containerisation) $ docker images
REPOSITORY                                          TAG      IMAGE ID     CREATED     SIZE
aws-bootcamp-cruddur-2023-again-frontend-react-js   latest   9a76f2ba97   7 min ago   1.18GB
aws-bootcamp-cruddur-2023-again-backend-flask       latest   464297df4e   1 hr ago    130MB
backend-flask                                       latest   12a857404d   1 hr ago    130MB
```
{% endcode %}

## 1.3 Workflow - Docker-compose

We now have 2 different images - one for the backend and the other for the frontend. We want to run one container for each, so there will a total of two containers running. As we continue to develop the app, we will have more containers in our app stack. When we have to manage multiple containers at the same time, manually managing them can be painful and time-consuming. This is where  `docker-compose` comes in to our rescue.&#x20;

We can compose multiple container definitions into a single yaml file and run it in one go. Our `docker-compose.yml` looks like below. The complete source code is [here](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/blob/01-00-app-containerisation/docker-compose.yml):&#x20;

* Once the file is ready, run:&#x20;
  * `docker compose -f "docker-compose.yml" up -d --build`
    * `-f` tag here means `--file`. Provide a file path after the tag.&#x20;
    * `up` is the command to run `docker-compose` using the specified file.&#x20;
    * `-d` is detached mode. The terminal in which you run `docker-compose` will be available for use again once the `docker-compose up` process is complete.
    * `--build`: build the image again in case the image for a container is not available or changes have been made to `Dockerfile`s.

<div data-full-width="true">

<figure><img src="../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

</div>

The image below is the result of `docker-compose up`. The two containers for the backend and the frontend are running at the same time. The endpoint url also returns the correct Cruddur page.&#x20;

<div data-full-width="true">

<figure><img src="../.gitbook/assets/image (7) (1) (1).png" alt=""><figcaption></figcaption></figure>

</div>

<div data-full-width="true">

<figure><img src="../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption><p>Our Cruddur application at the frontend endpoint when the Frontend container is running. </p></figcaption></figure>

</div>

In the Cruddur portal above:

* The API delivers the mock data.&#x20;
* There are no functions in the frontend UI.&#x20;
  * No buttons work because no feature has been implemented.

These are useful commands for `docker-compose` operations from now on.

* `docker compose -f "docker-compose.yml" up -d --build`
* `docker compose -f "docker-compose.yml" down`

This marks the end of our activity for the Week 1 live stream.&#x20;



## 2. Discussion

### Containers vs VMs (Virtual Machines)

Virtual Machine is a smart way to improve the use of your physical hardware resources. Some genius computer experts developed the concept and technologies from around 1960. Then about 20 years later, the container technology came into being. Both are technologies used to isolate environments, but how are they different?&#x20;

* VM: creates another whole computer inside your computer (which is the underlying host machine) using your computer's resources (memory, CPU, etc.). So this consume more resources of the host machine than containers do because every time a VM is created, it has to be equipped with everything including the OS (which is called a guest OS).
  * This could be pretty wasteful. If you travel to a different city or a country and stay there for a few nights, paying for a single room at a lodging is much cheaper and faster than renting the entire lodging service or building whole.
* Container: the docker containers are just isolated environments, but without replicating the  existing key resources repeatedly. It shares the host machine's OS, so it's much lighter and faster than the VM.&#x20;

<figure><img src="../.gitbook/assets/docker_technology.png" alt=""><figcaption><p>Docker containers (left) and VMs (right) both on a host machine</p></figcaption></figure>

### :maple\_leaf: Containerising your application

Putting your app into a container offers the following benefits:

* **Portability**: you can hand over your entire environment along with the configurations all set, so your team mates can jump in right away without going through the hassle of replicating your environment.
* **Compatibility**: dependencies such as Python libraries or Go packages, they get updated frequently. Let's say our project has a lot of dependencies and you forgot to specify the version for each dependency. Now it's our team mates' nightmare to deal with the version conflict to figure out which version of dependencies to install.

:maple\_leaf: **Guest instructors' insights**

* **Edith**: Makes testing easy - you can easily kill the container and start it over again, and again, and again.&#x20;
* **James**: as your team grows, the variety of dev environment will also grow - different OS, dependencies, if your app requires a compiler or not. Containerising the work environment reduces the pain stemming from dev env differences/incompatibilities

<figure><img src="../.gitbook/assets/main-week1 (1).png" alt=""><figcaption></figcaption></figure>

### More on --rm

`--rm` is really convenient because the container always cleans up after themselves. However..:

* sometimes we want to stop the docker container only without not removing it so we can rerun it and pick up something from it (such as logs).

On the other hand, if you stop a running the container but don't remove it, it will clog your machine, meaning it occupies your storage space. Below is what happens under the hood when you keep (re)building, running, and stopping containers. This happens often during the development & testing process.&#x20;

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

`<none>` are fragments of images. They are also called 'dangling' images or 'untagged' images. These are generated from orphaned images (that lose their tags) or multi-stage docker builds. So what to do about it?&#x20;

* `docker system prune -f`
  * You can clean all the dangling images by running this command.&#x20;
  * `-f` here means `--force`. Be sure to know what you are doing. You don't necessarily always want to clean up your storage with this force command.&#x20;

<figure><img src="../.gitbook/assets/image (41).png" alt=""><figcaption><p>Do you see <span data-gb-custom-inline data-tag="emoji" data-code="1f440">👀</span>? The dangling images were occupying 1.869 GB of space! Whoa!</p></figcaption></figure>

