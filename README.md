# Docker_basics

## Contents

1. [Docker commands](#docker-commands)
   - [Images](#images)
   - [Running containers](#running-containers)
   - [Managing containers](#managing-containers)
   - [Cleaning up](#cleaning-up)
   - [System information](#system-information)
2. [The Dockerfile](#the-dockerfile)
   - [The app: index.js](#the-app-indexjs)
   - [How the Dockerfile works](#how-the-dockerfile-works)
   - [Build the image](#build-the-image)
   - [Run the web server](#run-the-web-server)
   - [Stop the server](#stop-the-server)
   - [Earlier version: a one-line script](#earlier-version-a-one-line-script)

---

## Docker commands

An **image** is a read-only template, like a recipe. A **container** is a running (or stopped) instance of an image. You can start many containers from one image.

Most commands accept a container **name** or **ID**. You only need to type the first few characters of an ID, for example `docker stop 3f2a`.

### Images

| Command | What it does |
| --- | --- |
| `docker pull node:20` | Downloads an image from Docker Hub without running it. If you leave out the tag, Docker pulls `latest`. |
| `docker search node` | Searches Docker Hub for images with that name. Look for the **OFFICIAL** column. |
| `docker build . -t my-node:1.0` | Builds an image from the `Dockerfile` in the current folder and names it. See [Build the image](#build-the-image). |
| `docker image ls` | Lists the images on your machine. `docker images` does the same thing. |
| `docker history my-node` | Shows the layers of an image: which instruction created each layer and how big it is. |
| `docker tag my-node:1.0 my-node:latest` | Gives an existing image an extra name or tag. It does not copy the image. |
| `docker rmi my-node:1.0` | Removes an image. It fails if a container (even a stopped one) still uses it. |

### Running containers

`docker run` creates a new container from an image and starts it. If the image is not on your machine, Docker pulls it first.

```sh
docker run node:20
```

This runs the image's default command and the container stops when that command finishes. Every `docker run` creates a **new** container. To start an existing one again, use `docker start`.

#### Flags for `docker run`

| Flag | Short | What it does |
| --- | --- | --- |
| `--interactive` | `-i` | Keeps STDIN open, so the container can read what you type. |
| `--tty` | `-t` | Gives the container a terminal (TTY), so you get a prompt, colours and line editing. |
| `--detach` | `-d` | Runs the container in the background and prints its ID. Your terminal stays free. |
| `--name my-box` | | Gives the container a name. Without it, Docker picks a random name like `quirky_turing`. |
| `--rm` | | Deletes the container automatically when it stops. Good for one-off runs. |
| `--init` | | Runs a small init process as PID 1 so `Ctrl+C` and `docker stop` work properly. See [Run the web server](#run-the-web-server). |
| `--publish 8080:3000` | `-p` | Maps port `8080` on your machine to port `3000` in the container. |
| `--env KEY=value` | `-e` | Sets an environment variable inside the container. |
| `--volume ./data:/app/data` | `-v` | Mounts a folder from your machine into the container, so files survive after it is deleted. |

Short flags can be combined, so `-it` is `-i -t` and `-dit` is `-d -i -t`.

#### `-it`: interactive terminal

Use `-it` when you want to type into the container. `-i` alone has no prompt, and `-t` alone shows a prompt that does not read your input, so you almost always use them together.

```sh
docker run -it node:20          # opens the Node.js REPL
docker run -it node:20 bash     # opens a bash shell instead of the default command
```

Type `exit` (or press `Ctrl+D`) to leave. The container stops when you exit.

To leave **without** stopping the container, press `Ctrl+P` then `Ctrl+Q`.

#### `-dit`: interactive terminal in the background

`-d` on its own runs the default command in the background. If that command is a shell, it exits at once because nothing is connected to it. Adding `-it` keeps the shell open, so the container keeps running in the background:

```sh
docker run -dit --name my-box node:20 bash
docker exec -it my-box bash     # open a second shell inside it
docker attach my-box            # connect to the original shell
```

### Managing containers

| Command | What it does |
| --- | --- |
| `docker ps` | Lists **running** containers. |
| `docker ps --all` | Lists **all** containers, including stopped ones. Short form: `docker ps -a`. |
| `docker stop my-box` | Stops a container politely: sends `SIGTERM`, waits 10 seconds, then sends `SIGKILL`. |
| `docker kill my-box` | Stops a container at once with `SIGKILL`. Use it when `stop` hangs. |
| `docker start my-box` | Starts a stopped container again. Add `-ai` to attach to it interactively. |
| `docker restart my-box` | Stops and then starts a container. |
| `docker pause my-box` | Freezes all processes in the container. They keep their memory but use no CPU. |
| `docker unpause my-box` | Resumes a paused container from where it stopped. |
| `docker exec -it my-box bash` | Runs a command inside a **running** container. Here it opens a shell. |
| `docker attach my-box` | Connects your terminal to the container's main process. |
| `docker logs my-box` | Shows the output of a container. Add `-f` to keep following new output. |
| `docker inspect my-box` | Shows the full configuration of a container or image as JSON. |
| `docker rm my-box` | Removes a stopped container. Add `-f` to stop and remove a running one. |

### Cleaning up

| Command | What it does |
| --- | --- |
| `docker container prune` | Removes **all** stopped containers. It asks for confirmation first. |
| `docker image prune` | Removes dangling images (old layers with no name or tag). Add `-a` to remove every image not used by a container. |
| `docker system prune` | Removes stopped containers, unused networks, dangling images and build cache in one go. |

### System information

| Command | What it does |
| --- | --- |
| `docker version` | Shows the version of the Docker client and the Docker engine. |
| `docker info` | Shows system-wide details: number of containers and images, storage driver, CPUs, memory and more. |
| `docker --help` | Lists all commands. Use `docker <command> --help` for help with one command, for example `docker run --help`. |

---

## The Dockerfile

A `Dockerfile` is a text file with the steps Docker follows to build an image. This project packs a small Node.js web server into an image.

```dockerfile
FROM node:20

COPY index.js index.js

CMD [ "node", "index.js"]
```

### The app: index.js

```js
const http = require("http");

http
  .createServer(function(request, response) {
    console.log("request received");
    response.end("omg hi", "utf-8");
  })
  .listen(3000);
console.log("server started");
```

- It uses Node's built-in `http` module, so there is nothing to install (no `package.json` or `npm install`).
- It starts a web server that listens on port **3000** inside the container.
- For every request, it logs `request received` and replies with `omg hi`.
- When the server starts, it logs `server started`.

### How the Dockerfile works

Each line is an **instruction**, and Docker runs them from top to bottom:

- `FROM node:20` uses the official Node.js 20 image as the base. Every Dockerfile starts with `FROM`.
- `COPY index.js index.js` copies `index.js` from your project folder (the build context) into the image. The first path is on your machine; the second is inside the image. There is no `WORKDIR`, so the file ends up at `/index.js`.
- `CMD [ "node", "index.js"]` sets the default command a container runs when it starts: it runs the server.

`CMD` runs when you **start a container** (`docker run`), not when you **build the image** (`docker build`). To run something during the build, such as installing packages, use `RUN` instead.

**Layers and caching:** An image is made of layers. Instructions that change files (`RUN`, `COPY`, `ADD`) each add a new layer; others such as `CMD` only change the image's settings. Docker caches every step, so when you change a line, it rebuilds only that step and the steps after it. Here, when you edit `index.js`, Docker reuses the cached `node:20` base and rebuilds only from `COPY` down. Put lines that change often (like copying your source code) near the bottom to keep builds fast.

You can see the layers of your image with `docker history my-node-app:3`.

**Tip:** Most Dockerfiles add `WORKDIR /app` after `FROM`. It creates the folder and makes it the current directory, so your files do not land in the root (`/`) of the container.

### Build the image

From the project folder (where the `Dockerfile` is), run:

```sh
docker build . -t my-node-app:3
```

- `.` is the build context: the folder Docker sends to the build. Docker looks for the `Dockerfile` here, and `COPY` can only copy files from this folder.
- `-t my-node-app:3` gives the image a name (`my-node-app`) and a tag (`3`). Tags are often version numbers, so each new build can get a new tag.

If you leave out the tag (`-t my-node-app`), Docker uses the tag `latest`.

Check that the image exists:

```sh
docker image ls
```

**Note:** `docker run my-node-app` looks for `my-node-app:latest`. If you built the image with a different tag, use the same tag when you run it, for example `my-node-app:3`.

### Run the web server

```sh
docker run --init --publish 3000:3000 --rm --name na my-node-app:3
```

| Part | What it does |
| --- | --- |
| `docker run` | Creates a new container from an image and starts it. |
| `--init` | Starts a tiny init program ([tini](https://github.com/krallin/tini)) as the first process (PID 1) in the container, and runs `node index.js` under it. See below for why. |
| `--publish 3000:3000` | Maps a port, in the form `HOST:CONTAINER`. Requests to port `3000` on your machine go to port `3000` in the container. Short form: `-p 3000:3000`. |
| `--rm` | Deletes the container automatically when it stops, so stopped containers do not pile up. |
| `--name na` | Names the container `na`, so you can use `docker stop na` instead of an ID. Names must be unique: while `na` exists, another `docker run --name na` fails. |
| `my-node-app:3` | The image (`my-node-app`) and tag (`3`) to start the container from. |

The terminal shows `server started`. Open <http://localhost:3000> in a browser, or run:

```sh
curl localhost:3000
```

You get `omg hi`, and the container logs `request received`. A browser often logs it twice, because it also asks for `/favicon.ico`.

The server keeps running and holds your terminal. To get your terminal back, add `--detach` (`-d`) to run it in the background, and use `docker logs na` to see its output.

#### Why `--publish`?

A container has its own network. The server listens on port `3000` **inside** the container, but your machine cannot reach it until you publish the port. Without `--publish`, <http://localhost:3000> does not connect.

The two numbers do not have to match. With `--publish 8080:3000`, you open <http://localhost:8080> and Docker forwards it to port `3000` in the container.

#### Why `--init`?

The first process in a container gets PID 1, and Linux treats PID 1 specially: it ignores signals such as `SIGINT` (`Ctrl+C`) and `SIGTERM` (`docker stop`) unless the program handles them itself. `index.js` does not handle them, so without `--init`:

- `Ctrl+C` does nothing.
- `docker stop` waits 10 seconds, then kills the container with `SIGKILL`.

With `--init`, tini is PID 1. It passes signals on to Node.js, so `Ctrl+C` and `docker stop` stop the server at once. Tini also cleans up finished child processes ("zombie" processes).

### Stop the server

- Press `Ctrl+C` in the terminal where it runs (this works because of `--init`), or
- From another terminal, run `docker stop na`.

Because of `--rm`, the container is deleted when it stops, so `docker ps --all` no longer shows it and you can use the name `na` again.

### Earlier version: a one-line script

The first version of the Dockerfile did not copy any files. It ran a one-line script and exited:

```dockerfile
FROM node:20
CMD [ "node", "-e", "console.log(\"hi omg lol\")"]
```

```sh
docker build . -t my-node:version
docker run my-node:version     # prints "hi omg lol", then the container stops
```

`node -e` runs the JavaScript given as a string. Because the script ends right away, the container stops too. A container runs only as long as its main process runs, which is why the web server version keeps running until you stop it.
