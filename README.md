# Docker_basics

## Contents

1. [Project structure](#project-structure)
2. [Docker commands](#docker-commands)
   - [Images](#images)
   - [Running containers](#running-containers)
   - [Managing containers](#managing-containers)
   - [Processes inside a container: ps aux](#processes-inside-a-container-ps-aux)
   - [Cleaning up](#cleaning-up)
   - [System information](#system-information)
3. [App 1: build-a-nodejs-app](#app-1-build-a-nodejs-app)
   - [The app: index.js](#the-app-indexjs)
   - [How the Dockerfile works](#how-the-dockerfile-works)
   - [Working directory: WORKDIR](#working-directory-workdir)
   - [Running as a non-root user](#running-as-a-non-root-user)
   - [Build the image](#build-the-image)
   - [Run the web server](#run-the-web-server)
   - [Stop the server](#stop-the-server)
   - [Earlier version: a one-line script](#earlier-version-a-one-line-script)
4. [App 2: build-a-more-complicated-nodejs-app](#app-2-build-a-more-complicated-nodejs-app)
   - [The app: Fastify](#the-app-fastify)
   - [How this Dockerfile works](#how-this-dockerfile-works)
   - [Installing dependencies: npm ci](#installing-dependencies-npm-ci)
   - [Faster builds: copy package files first](#faster-builds-copy-package-files-first)
   - [Leaving files out: .dockerignore](#leaving-files-out-dockerignore)
   - [Build and run the Fastify app](#build-and-run-the-fastify-app)
   - [Why host 0.0.0.0?](#why-host-0000)
   - [App 1 and App 2 compared](#app-1-and-app-2-compared)

---

## Project structure

Each app has its own folder, with its own `Dockerfile`. Build and run each one from inside its folder.

```
my_project/
├── README.md
├── .gitignore
├── build-a-nodejs-app/                  # App 1: web server with no dependencies
│   ├── Dockerfile
│   └── index.js
└── build-a-more-complicated-nodejs-app/ # App 2: Fastify server with npm dependencies
    ├── .dockerignore
    ├── Dockerfile
    ├── index.js
    ├── package.json
    └── package-lock.json
```

`node_modules/` is not committed (see `.gitignore`). In App 2, the image installs its own copy during the build.

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
| `docker top my-box` | Lists the processes running in a container, from the outside. See [ps aux](#processes-inside-a-container-ps-aux). |
| `docker attach my-box` | Connects your terminal to the container's main process. |
| `docker logs my-box` | Shows the output of a container. Add `-f` to keep following new output. |
| `docker inspect my-box` | Shows the full configuration of a container or image as JSON. |
| `docker rm my-box` | Removes a stopped container. Add `-f` to stop and remove a running one. |

### Processes inside a container: ps aux

`ps aux` is a Linux command, not a Docker command. It lists every process that is running. Run it inside a container with `docker exec` to see what the container is doing and **which user** each process runs as.

```sh
docker exec na ps aux
```

The letters are three options:

- `a`: show processes of **all** users, not only yours.
- `u`: use the **user-oriented** format, which adds the `USER`, `%CPU` and `%MEM` columns.
- `x`: also show processes that have no terminal, such as servers.

With App 1's web server running (see [Run the web server](#run-the-web-server)), the output looks like this:

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
lolcat       1  0.4  0.0    816     4 ?        Ss   13:08   0:00 /sbin/docker-init -- docker-entrypoint.sh node index.js
lolcat       7  1.8  0.4 464248 40056 ?        Sl   13:08   0:00 node index.js
lolcat      15 25.0  0.0   8164  3652 ?        Rs   13:08   0:00 ps aux
```

| Column | Meaning |
| --- | --- |
| `USER` | The user that runs the process. Here it is `lolcat`, because of `USER lolcat` in the Dockerfile. Without it, this column says `root`. |
| `PID` | The process ID. Inside a container, numbering starts at 1. |
| `%CPU`, `%MEM` | How much CPU and memory the process uses. |
| `VSZ`, `RSS` | Virtual memory and real (resident) memory, in KB. |
| `TTY` | The terminal the process is attached to. `?` means none. |
| `STAT` | The state: `R` running, `S` sleeping (waiting for something, like a request), `s` session leader, `l` multi-threaded. |
| `START`, `TIME` | When the process started, and how much CPU time it has used in total. |
| `COMMAND` | The command that started the process. |

Things you can see in this output:

- **PID 1 is `docker-init`**, not Node.js, because of `--init`. Node.js runs under it. Without `--init`, `node` is PID 1. See [Why `--init`?](#why---init).
- **`docker-entrypoint.sh`** is a small script that the `node` images run first (their `ENTRYPOINT`). It passes your `CMD` (`node index.js`) on and then hands over to it, which is why PID 7 shows only `node index.js`.
- **Everything runs as `lolcat`**, not `root`. See [Running as a non-root user](#running-as-a-non-root-user). In App 2, this column says `node`.
- The container runs very few processes, only the ones it needs. The `ps aux` line is the command you just ran.

`docker top na` shows the same list from outside the container. It works even when the image has no `ps` command; small images such as `node:20-slim` or `alpine` variants may not include it.

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

## App 1: build-a-nodejs-app

A `Dockerfile` is a text file with the steps Docker follows to build an image. This app, in the `build-a-nodejs-app/` folder, packs a small Node.js web server into an image.

```dockerfile
FROM node:20

RUN useradd -ms /bin/bash lolcat

USER lolcat

WORKDIR /home/lolcat/code

COPY --chown=lolcat index.js .

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
- `RUN useradd -ms /bin/bash lolcat` runs a Linux command during the build. It creates a new user called `lolcat` with the home folder `/home/lolcat`. See [Create your own user](#create-your-own-user-this-dockerfile).
- `USER lolcat` makes the rest of the build, and the container, run as `lolcat` instead of `root`. See [Running as a non-root user](#running-as-a-non-root-user).
- `WORKDIR /home/lolcat/code` creates the folder `/home/lolcat/code` and makes it the current directory for the steps after it and for the container. See [Working directory: WORKDIR](#working-directory-workdir).
- `COPY --chown=lolcat index.js .` copies `index.js` from your project folder (the build context) into the image. The first path is on your machine; the second, `.`, means "the current directory" inside the image, which is `/home/lolcat/code` because of `WORKDIR`. So the file ends up at `/home/lolcat/code/index.js`. `--chown=lolcat` makes `lolcat` the owner of the copied file. See [File ownership: COPY --chown](#file-ownership-copy---chown).
- `CMD [ "node", "index.js"]` sets the default command a container runs when it starts: it runs the server. A relative path is enough, because the container starts in the `WORKDIR`.

`CMD` runs when you **start a container** (`docker run`), not when you **build the image** (`docker build`). To run something during the build, such as installing packages, use `RUN` instead.

**Layers and caching:** An image is made of layers. Instructions that change files (`RUN`, `COPY`, `ADD`) each add a new layer; others such as `CMD` only change the image's settings. Docker caches every step, so when you change a line, it rebuilds only that step and the steps after it. Here, `RUN useradd` adds a layer (the new user's entries in `/etc/passwd` and its home folder), and `COPY` adds another. When you edit `index.js`, Docker reuses the cached `node:20` base, `RUN`, `USER` and `WORKDIR` steps (the build output shows `CACHED`), and rebuilds only from `COPY` down. Put lines that change often (like copying your source code) near the bottom to keep builds fast.

You can see the layers of your image with `docker history my-node-app:3`.

### Working directory: WORKDIR

`WORKDIR` sets the **current directory** inside the image, like running `cd` in a terminal. If the folder does not exist, Docker creates it.

```dockerfile
WORKDIR /home/lolcat/code
```

Without a `WORKDIR`, the current directory is `/` (the root of the file system), unless the base image sets one. `node:20` does not.

#### What WORKDIR affects

Everything **after** it uses the new directory:

| Where | Effect in this Dockerfile |
| --- | --- |
| `COPY` and `ADD` | A relative destination is relative to the `WORKDIR`. `COPY index.js .` puts the file at `/home/lolcat/code/index.js`. |
| `RUN` | Commands run inside the `WORKDIR`. For example, `RUN npm install` would install into `/home/lolcat/code/node_modules`. |
| `CMD` | The container starts in the `WORKDIR`, so `node index.js` finds the file. |
| `docker exec` | Commands you run in the container also start there. |

Check it while the server runs:

```sh
docker exec na pwd      # prints: /home/lolcat/code
docker exec na ls -l    # lists index.js, without typing the full path
```

#### Before and after

Without `WORKDIR`, every path has to be written in full:

```dockerfile
COPY --chown=lolcat index.js /home/lolcat/code/index.js
CMD [ "node", "/home/lolcat/code/index.js"]
```

With `WORKDIR`, the folder is written once and the other lines get shorter:

```dockerfile
WORKDIR /home/lolcat/code
COPY --chown=lolcat index.js .
CMD [ "node", "index.js"]
```

If you move the app to another folder later, you only change the `WORKDIR` line.

#### Why not `RUN cd`?

Each `RUN` runs in a new shell, so a `cd` is forgotten as soon as that step ends:

```dockerfile
RUN cd /home/lolcat/code   # has no effect on the next lines
RUN ls                     # still runs in /
```

Use `WORKDIR` instead. It is saved in the image's settings, so every later step and the container itself use it.

#### More about WORKDIR

- **Use absolute paths.** `WORKDIR` can be relative, but then it builds on the previous one: `WORKDIR /a` then `WORKDIR b` gives `/a/b`. An absolute path like `/home/lolcat/code` is clearer.
- **Who owns the folder.** If `WORKDIR` creates the folder after a `USER` line, BuildKit (the default builder in Docker Desktop and current Docker) makes that user the owner. Here `ls -ld /home/lolcat/code` shows `lolcat lolcat`. The old builder (`DOCKER_BUILDKIT=0`) makes `root` the owner instead. This matters in App 2, where `npm ci` must create `node_modules` inside the `WORKDIR`.
- **Override it at run time.** `docker run --workdir /tmp my-node-app:3` (short form `-w`) starts the container in another folder. Here that breaks the app, because `node index.js` would look for `/tmp/index.js`.

### Running as a non-root user

By default, a container runs as `root`. The app does not need root to answer web requests, and if someone breaks into it, root gives them much more power: they can change any file in the container and have a better chance of escaping it. Running as a normal user limits the damage.

The `USER` instruction picks the user. It affects everything after it:

- `RUN` steps later in the Dockerfile run as that user.
- The `CMD`, and anything you start with `docker exec`, run as that user.

The user must already exist in the image. There are two ways to get one.

#### Option 1: the built-in node user

The official `node` images already have a user called `node` (home folder `/home/node`), so this is enough:

```dockerfile
FROM node:20
USER node
COPY index.js /home/node/code/index.js
CMD [ "node", "/home/node/code/index.js"]
```

This was the previous version of App 1's Dockerfile, and [App 2](#app-2-build-a-more-complicated-nodejs-app) uses this user too.

#### Create your own user (this Dockerfile)

Most base images do not come with a ready-made user, so you create one with `RUN useradd`:

```dockerfile
RUN useradd -ms /bin/bash lolcat
USER lolcat
```

| Part | What it does |
| --- | --- |
| `useradd` | The Linux command that creates a user. It also creates a group with the same name. |
| `-m` | Creates the user's home folder, `/home/lolcat`. Without it, the folder does not exist. |
| `-s /bin/bash` | Sets the user's login shell to bash. The default for `useradd` on Debian (which `node:20` is built on) is `/bin/sh`. |
| `-ms` | `-m` and `-s` combined. |
| `lolcat` | The name of the new user. |

The order matters:

- `RUN useradd` must come **before** `USER lolcat`, because only `root` can create users.
- `USER lolcat` must come **after** `RUN useradd`. If the user does not exist, the container fails to start with an error like `unable to find user lolcat`.

Each user has a number (UID). The `node` user already has UID `1000`, so `lolcat` gets the next free one, `1001`.

**Note:** `useradd` is for Debian and Ubuntu based images like `node:20`. Alpine images (`node:20-alpine`) use `adduser` instead.

#### Check which user runs the app

While the server runs:

```sh
docker exec na whoami     # prints: lolcat
docker exec na id         # prints: uid=1001(lolcat) gid=1001(lolcat) groups=1001(lolcat)
docker exec na ps aux     # the USER column says lolcat
```

See [Processes inside a container: ps aux](#processes-inside-a-container-ps-aux) for how to read the `ps aux` output.

To get a root shell for debugging, override the user: `docker exec -it --user root na bash`.

#### File ownership: COPY --chown

`USER` does **not** change who owns copied files. Without `--chown`, `COPY` always creates files owned by `root`, even after `USER lolcat`, and even in `lolcat`'s home folder. Then `lolcat` can read `index.js` but cannot change it.

That is why this Dockerfile uses `--chown`:

```dockerfile
COPY --chown=lolcat index.js .
```

| Form | Owner | Group |
| --- | --- | --- |
| `COPY index.js ...` | `root` | `root` |
| `COPY --chown=lolcat index.js ...` | `lolcat` | the group with the same number as `lolcat`'s UID (`1001`), which here is the `lolcat` group |
| `COPY --chown=lolcat:lolcat index.js ...` | `lolcat` | `lolcat`, named explicitly |

`--chown=lolcat:lolcat` is the clearer form, because it names the group too. Both give the same result here, because `useradd` created a `lolcat` group with the same number as the user.

Check the owners while the server runs:

```sh
docker exec na ls -l /home/lolcat/code/index.js   # owner and group: lolcat lolcat
docker exec na ls -ld /home/lolcat/code           # the code folder that WORKDIR created
```

Now `lolcat` owns `index.js` and can change it. This server only reads its own file, so it works either way. Ownership starts to matter when the app writes files, for example logs, uploads or a `node_modules` folder that `npm ci` creates, as in [App 2](#app-2-build-a-more-complicated-nodejs-app).

**Note:** `--chown` only works with user and group names that exist in the image, so `RUN useradd` must come before this `COPY`. It also only works for Linux containers.

### Build the image

Go into the app's folder (where its `Dockerfile` is) and build:

```sh
cd build-a-nodejs-app
docker build . -t my-node-app:3
```

- `.` is the build context: the folder Docker sends to the build. Docker looks for the `Dockerfile` here, and `COPY` can only copy files from this folder.
- You can also build from the project root by giving the folder instead of `.`: `docker build build-a-nodejs-app -t my-node-app:3`.
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

---

## App 2: build-a-more-complicated-nodejs-app

App 1 used only Node's built-in modules. Real apps use packages from npm, so the image must install them during the build. This app, in the `build-a-more-complicated-nodejs-app/` folder, uses the [Fastify](https://fastify.dev) web framework.

```dockerfile
FROM node:20

USER node

WORKDIR /home/node/code

COPY --chown=node package*.json ./

RUN npm ci

COPY --chown=node . .

CMD [ "node", "index.js"]
```

### The app: Fastify

```js
// this is the sample app from fastify.dev

// Require the framework and instantiate it
const fastify = require("fastify")({ logger: true });

// Declare a route
fastify.get("/", function handler(request, reply) {
  reply.send({ hello: "world" });
});

// Run the server!
fastify.listen({ port: 8080, host: "0.0.0.0" }, (err) => {
  if (err) {
    fastify.log.error(err);
    process.exit(1);
  }
});
```

- `require("fastify")` loads Fastify from `node_modules`, so it must be installed first. `package.json` lists it as a dependency (`"fastify": "^5.12.7"`), and `package-lock.json` records the exact versions of Fastify and everything it needs.
- `{ logger: true }` turns on Fastify's logger. It prints one line of JSON for each event, for example when the server starts and when each request comes in and is answered.
- `fastify.get("/", ...)` adds a route: a `GET` request to `/` gets the JSON reply `{"hello":"world"}`.
- `fastify.listen({ port: 8080, host: "0.0.0.0" })` starts the server on port **8080**, on all network interfaces. See [Why host 0.0.0.0?](#why-host-0000).
- If the server cannot start (for example, the port is already in use), it logs the error and exits with code `1`.

### How this Dockerfile works

- `FROM node:20`: the same base image as App 1.
- `USER node`: uses the `node` user that the image already has, instead of creating one. See [Option 1: the built-in node user](#option-1-the-built-in-node-user).
- `WORKDIR /home/node/code`: creates the app folder, owned by `node`, and makes it the current directory. See [Working directory: WORKDIR](#working-directory-workdir).
- `COPY --chown=node package*.json ./`: copies **only** `package.json` and `package-lock.json` (`*` matches anything). When `COPY` has more than one source, the destination must be a folder ending in `/`, so it is `./` instead of `.`.
- `RUN npm ci`: installs the dependencies into `/home/node/code/node_modules`. It runs as `node`, so `node` owns `node_modules`. See [Installing dependencies: npm ci](#installing-dependencies-npm-ci).
- `COPY --chown=node . .`: copies the rest of the app folder (the whole build context, except what `.dockerignore` leaves out) into the `WORKDIR`.
- `CMD [ "node", "index.js"]`: starts the server.

`npm ci` can create `node_modules` only because `node` owns the `WORKDIR` folder. If `root` owned it, `npm ci` would fail with a permission error (`EACCES`).

### Installing dependencies: npm ci

`npm ci` ("clean install") is the npm command made for builds and automated setups:

| | `npm ci` | `npm install` |
| --- | --- | --- |
| Needs `package-lock.json` | Yes. It fails without one. | No. It creates one if missing. |
| Versions | Installs exactly the versions in `package-lock.json`. | Can install newer versions allowed by `package.json` (like `^5.12.7`) and update the lock file. |
| If `package.json` and the lock file disagree | Fails with an error. | Updates the lock file. |
| Existing `node_modules` | Deletes it first. | Updates it. |

With `npm ci`, every build installs the same versions you tested with on your machine. Use `npm install` on your machine when you add or update packages, and commit the updated `package-lock.json`.

### Faster builds: copy package files first

The Dockerfile copies the app in **two** steps on purpose:

```dockerfile
COPY --chown=node package*.json ./   # 1. only the dependency list
RUN npm ci                           # 2. install (slow)
COPY --chown=node . .                # 3. the rest of the code
```

Docker reuses a cached step until something it depends on changes. You edit `index.js` much more often than you change dependencies:

- **You edit `index.js`:** steps 1 and 2 show `CACHED`, and only step 3 runs again. The build takes a second or two.
- **You change `package.json` or `package-lock.json`:** step 1 changes, so `npm ci` runs again.

If the Dockerfile copied everything with one `COPY . .` before `RUN npm ci`, any change to any file would run `npm ci` again, and every build would download all packages.

### Leaving files out: .dockerignore

`.dockerignore` lists files and folders that Docker does **not** send to the build, like `.gitignore` does for git:

```
node_modules/
.git/
```

- **`node_modules/`:** If you ran `npm install` on your machine, you have a local `node_modules` folder. Without this line, `COPY . .` would copy it over the one that `npm ci` just installed in the image. Your local copy was made for macOS, while the container runs Linux, so packages with native code could break. It also makes the build context much bigger and slower to send.
- **`.git/`:** The git history is not needed to run the app.

`COPY . .` copies everything else, so the image also gets the `Dockerfile` and `.dockerignore` itself. They are harmless, but you can add them to `.dockerignore` to keep the image clean.

### Build and run the Fastify app

```sh
cd build-a-more-complicated-nodejs-app
docker build . -t my-fastify-app:1
docker run --init --publish 8080:8080 --rm --name fa my-fastify-app:1
```

The flags are the same as in [Run the web server](#run-the-web-server). This time, the port is `8080`, because that is the port the app listens on.

The logger prints JSON lines like these:

```
{"level":30,"time":1793279323175,"pid":7,"hostname":"d1e3da1794af","msg":"Server listening at http://127.0.0.1:8080"}
{"level":30,"time":1793279323175,"pid":7,"hostname":"d1e3da1794af","msg":"Server listening at http://172.17.0.2:8080"}
```

- `level` `30` means `info`.
- `pid` `7` is the Node.js process. It is not `1`, because of `--init`.
- `hostname` is the container ID.
- The two `listening` lines show the two addresses inside the container: `127.0.0.1` (the container itself) and `172.17.0.2` (the container's address on the Docker network).

Open <http://localhost:8080>, or run:

```sh
curl localhost:8080      # {"hello":"world"}
```

Each request adds an `incoming request` line and a `request completed` line to the log. Stop the server with `Ctrl+C` or `docker stop fa`.

### Why host 0.0.0.0?

`host` decides which network addresses the server accepts connections on:

| `host` | Accepts connections from |
| --- | --- |
| `localhost` or `127.0.0.1` | Only the container itself. |
| `0.0.0.0` | All network interfaces, including the Docker network. |

With `--publish`, Docker sends your request to the container's address on the Docker network (`172.17.0.2` above), **not** to its `localhost`. Fastify listens only on `localhost` by default, so without `host: "0.0.0.0"`, the published port does not work and <http://localhost:8080> fails to connect.

App 1 did not need this, because Node's `http` `.listen(3000)` listens on all addresses by default.

### App 1 and App 2 compared

| | App 1 | App 2 |
| --- | --- | --- |
| Framework | Node's built-in `http` | Fastify, from npm |
| Dependencies | None | `package.json` and `package-lock.json`, installed with `npm ci` |
| User | `lolcat`, created with `useradd` | `node`, already in the image |
| What is copied | `index.js` only | Package files first, then the whole folder |
| `.dockerignore` | Not needed | Leaves out `node_modules/` and `.git/` |
| Port | `3000` | `8080`, on host `0.0.0.0` |
| Reply | `omg hi` (text) | `{"hello":"world"}` (JSON) |
| Logs | Plain text with `console.log` | JSON lines from Fastify's logger |
