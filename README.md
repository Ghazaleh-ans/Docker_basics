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
   - [Security scanning: docker scout](#security-scanning-docker-scout)
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
5. [App 3: alpine-linux](#app-3-alpine-linux)
   - [What is Alpine Linux?](#what-is-alpine-linux)
   - [Alpine and Debian compared](#alpine-and-debian-compared)
   - [Benefits of Alpine](#benefits-of-alpine)
   - [Things that work differently on Alpine](#things-that-work-differently-on-alpine)
   - [When to use Alpine, slim or the full image](#when-to-use-alpine-slim-or-the-full-image)
   - [Build and run the Alpine app](#build-and-run-the-alpine-app)
   - [Version 2: install Node.js on plain Alpine](#version-2-install-nodejs-on-plain-alpine)
   - [Version 1 and Version 2 compared](#version-1-and-version-2-compared)
   - [Which version to use](#which-version-to-use)
6. [App 4: multi-stage-builds](#app-4-multi-stage-builds)
   - [What is a multi-stage build?](#what-is-a-multi-stage-build)
   - [How this Dockerfile works](#how-this-dockerfile-works-1)
   - [What ends up in the final image](#what-ends-up-in-the-final-image)
   - [Build and run the multi-stage app](#build-and-run-the-multi-stage-app)
   - [Single-stage and multi-stage compared](#single-stage-and-multi-stage-compared)
   - [Watch out: native modules](#watch-out-native-modules)
   - [Making it smaller and simpler](#making-it-smaller-and-simpler)
   - [When to use multi-stage builds](#when-to-use-multi-stage-builds)
7. [App 5: distroless](#app-5-distroless)
   - [What is distroless?](#what-is-distroless)
   - [How the distroless Dockerfile works](#how-the-distroless-dockerfile-works)
   - [Build and run the distroless app](#build-and-run-the-distroless-app)
   - [Living without a shell](#living-without-a-shell)
   - [Problem 1: the app runs as root](#problem-1-the-app-runs-as-root)
   - [Problem 2: no .dockerignore](#problem-2-no-dockerignore)
   - [Distroless compared with the other apps](#distroless-compared-with-the-other-apps)
   - [When to use distroless](#when-to-use-distroless)
8. [App 6: static-asset-project](#app-6-static-asset-project)
   - [The app: an Astro website](#the-app-an-astro-website)
   - [How the static site Dockerfile works](#how-the-static-site-dockerfile-works)
   - [What is nginx?](#what-is-nginx)
   - [Build and run the static site](#build-and-run-the-static-site)
   - [What is inside the final image](#what-is-inside-the-final-image)
   - [Your Dockerfile and solution.Dockerfile](#your-dockerfile-and-solutiondockerfile)
   - [Serve the site with a bind mount](#serve-the-site-with-a-bind-mount)
   - [Bind mounts](#bind-mounts)
   - [Improving the static site build](#improving-the-static-site-build)
   - [When to serve static files with nginx](#when-to-serve-static-files-with-nginx)
9. [App 7: docker-volume](#app-7-docker-volume)
   - [The app: a counter that saves to a file](#the-app-a-counter-that-saves-to-a-file)
   - [The incrementor Dockerfile](#the-incrementor-dockerfile)
   - [Run the incrementor with a volume](#run-the-incrementor-with-a-volume)
   - [Environment variables: --env](#environment-variables---env)
   - [Volumes](#volumes)
   - [Managing volumes](#managing-volumes)
   - [Listing volumes: docker volume ls](#listing-volumes-docker-volume-ls)
   - [Volumes and non-root users](#volumes-and-non-root-users)
   - [Volumes, bind mounts and the container's own files](#volumes-bind-mounts-and-the-containers-own-files)

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
├── build-a-more-complicated-nodejs-app/ # App 2: Fastify server with npm dependencies
│   ├── .dockerignore
│   ├── Dockerfile
│   ├── index.js
│   ├── package.json
│   └── package-lock.json
├── alpine-linux/                        # App 3: App 2 on a small Alpine Linux base image
│   ├── .dockerignore
│   ├── Dockerfile
│   ├── index.js
│   ├── package.json
│   └── package-lock.json
├── multi-stage-builds/                  # App 4: install on Debian, run on Alpine
│   ├── .dockerignore
│   ├── Dockerfile
│   ├── index.js
│   ├── package.json
│   └── package-lock.json
├── distroless/                          # App 5: install on Debian, run on distroless
│   ├── Dockerfile
│   ├── index.js
│   ├── package.json
│   └── package-lock.json
├── static-asset-project/                # App 6: build a website with Node.js, serve it with nginx
│   ├── Dockerfile
│   ├── solution.Dockerfile
│   ├── astro.config.mjs, tailwind.config.mjs, tsconfig.json
│   ├── package.json
│   ├── package-lock.json
│   ├── public/                          # files copied as they are (favicon.svg)
│   └── src/                             # pages, layouts and components
└── docker-volume/                       # App 7: a counter that keeps its data in a volume
    ├── Dockerfile
    └── index.js
```

`node_modules/` is not committed (see `.gitignore`). In App 2 to App 5, the image installs its own copy during the build.

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
| `--env KEY=value` | `-e` | Sets an environment variable inside the container. See [Environment variables](#environment-variables---env). |
| `--volume ./data:/app/data` | `-v` | Mounts a folder from your machine into the container, so files survive after it is deleted. See [Bind mounts](#bind-mounts). |
| `--mount type=bind,source=...,target=...` | | The longer, clearer way to mount a folder. See [Bind mounts](#bind-mounts). |
| `--mount type=volume,src=...,target=...` | | Attaches a Docker-managed volume, for data that must outlive the container. See [Volumes](#volumes). |

Short flags can be combined, so `-it` is `-i -t` and `-dit` is `-d -i -t`.

#### `-it`: interactive terminal

Use `-it` when you want to type into the container. `-i` alone has no prompt, and `-t` alone shows a prompt that does not read your input, so you almost always use them together.

```sh
docker run -it node:20          # opens the Node.js REPL
docker run -it node:20 bash     # opens a bash shell instead of the default command
docker run -it node:20-alpine sh  # Alpine images have no bash, so use sh
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

`docker top na` shows the same list from outside the container. It works even when the image has no `ps` command, as in `node:20-slim`, or no shell at all, as in [distroless](#app-5-distroless).

On Alpine images, `ps` is the small BusyBox version. It ignores `aux` and always shows only four columns:

```
PID   USER     TIME  COMMAND
    1 node      0:00 /sbin/docker-init -- docker-entrypoint.sh node index.js
    8 node      0:00 node index.js
```

See [App 3: alpine-linux](#app-3-alpine-linux).

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

### Security scanning: docker scout

[Docker Scout](https://docs.docker.com/scout/) checks an image for **known security holes**. It makes a list of every package in the image (an SBOM, "software bill of materials"), then compares that list with public vulnerability databases.

Each known security hole has an ID called a **CVE** ("Common Vulnerabilities and Exposures"), for example `CVE-2024-5535`. Each CVE gets a **severity**:

| Letter | Severity | Meaning |
| --- | --- | --- |
| `C` | Critical | Fix now. Often easy to attack, with serious damage. |
| `H` | High | Fix soon. |
| `M` | Medium | Fix when you can. |
| `L` | Low | Small risk. |

The package list is sent to Docker's service to be checked, so you need an internet connection. Newer versions of Docker Scout ask you to sign in first with `docker login`.

#### docker scout quickview

```sh
docker scout quickview alpine:3.19.1
```

Shows a **short summary** of an image: how many vulnerabilities it has at each severity (for example `1C 5H 10M 2L`). It shows the same counts for the image's **base image**, and, if there is one, suggests a newer base image with fewer vulnerabilities. Use it first, to see whether an image has a problem at all.

`quickview` needs a recent Docker Scout. On this machine, Docker Scout is v0.6.0, which only has `cves` and fails with `unknown command: "quickview"`. Update Docker Desktop to get it, and check your version with `docker scout version`.

#### docker scout cves

```sh
docker scout cves alpine:3.19.1
```

Shows the **full list**: every vulnerable package, every CVE in it, and the version that fixes it. Use it after `quickview`, to see what to fix.

Result for `alpine:3.19.1`, measured on this project (shortened):

```
    ✓ Indexed 19 packages
    ✗ Detected 3 vulnerable packages with a total of 18 vulnerabilities

  1C    3H    5M    0L  openssl 3.1.4-r5
    ✗ CRITICAL CVE-2024-5535
      https://scout.docker.com/v/CVE-2024-5535?...
      Affected range : <3.1.6-r0
      Fixed version  : 3.1.6-r0
    ...

  0C    2H    1M    0L  musl 1.2.4_git20230717-r4
    ...

  0C    0H    4M    2L  busybox 1.36.1-r15
    ...

  LOW      | 2
  MEDIUM   | 10
  HIGH     | 5
  CRITICAL | 1
```

How to read it:

- `Indexed 19 packages`: Scout found 19 packages in the image.
- Each block starts with **one package** and its version, for example `openssl 3.1.4-r5`, and the number of CVEs in it by severity.
- `Affected range : <3.1.6-r0` means every version **before** `3.1.6-r0` has this hole.
- `Fixed version : 3.1.6-r0` is the first version without it. Every CVE here has a fixed version, so all 18 can be fixed by updating.
- The link opens a page about the CVE on Docker's website.
- The table at the end counts all CVEs by severity.

Useful options:

| Option | What it does |
| --- | --- |
| `--exit-code` (`-e`) | Exits with code `2` if vulnerabilities are found. In CI, this makes the job fail. |
| `--format sarif --output report.json` | Saves the report as a SARIF file, a format that GitHub and other tools can read. |
| `--details` | Prints more details for each CVE. |

You can also scan your own images, for example `docker scout cves my-fastify-app:alpine`.

#### Why does alpine:3.19.1 have 18 vulnerabilities?

An image is **frozen** when it is built. `alpine:3.19.1` was released in January 2024, so it contains the package versions from that time. Security fixes released later are in Alpine's package repository, but **not** in that image.

Measured on this project:

| Image | Vulnerabilities |
| --- | --- |
| `alpine:3.19.1` (an old patch release) | 18 (1 critical, 5 high, 10 medium, 2 low) |
| `alpine:3.19` (the newest 3.19 patch release, 3.19.9) | 4 (1 high, 1 medium, 2 low) |
| `alpine:3.19.1` + `RUN apk upgrade --no-cache` | **0** ("No vulnerable package detected") |

So, to get rid of known vulnerabilities:

- **Use the newest patch release.** Use the tag `alpine:3.19` (always the newest `3.19.x`) instead of an exact one like `3.19.1`. Even better, use a newer Alpine version that is still supported.
- **Update the packages during the build.** `RUN apk upgrade --no-cache` installs the fixed versions. Here it made the image bigger (15.2 MB instead of 7.7 MB), because the updated files are stored in a new layer on top of the old ones.
- **Rebuild your images regularly**, so they pick up new fixes from the base image.
- **Scan again after every change.** New CVEs are found all the time, so an image with 0 today can have some next month.

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

**Note:** `useradd` is for Debian and Ubuntu based images like `node:20`. Alpine images (`node:20-alpine`) do not have it (`useradd: not found`). Use `adduser` instead: `RUN adduser -D lolcat` creates `lolcat` with the home folder `/home/lolcat` and no password (`-D`). Its default shell is `/bin/sh`, because Alpine has no bash. See [App 3: alpine-linux](#app-3-alpine-linux).

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

---

## App 3: alpine-linux

This app, in the `alpine-linux/` folder, runs App 2 on Alpine Linux. It has two versions:

- **Version 1** (this section, up to [Build and run the Alpine app](#build-and-run-the-alpine-app)): App 2 with **one** change, the base image `node:20-alpine`.
- **Version 2** (the current `Dockerfile` in the folder): starts from plain `alpine:3.19` and installs Node.js itself. See [Version 2: install Node.js on plain Alpine](#version-2-install-nodejs-on-plain-alpine).

Version 1:

```dockerfile
FROM node:20-alpine

USER node

WORKDIR /home/node/code

COPY --chown=node package*.json ./

RUN npm ci

COPY --chown=node . .

CMD [ "node", "index.js"]
```

`index.js`, `package.json`, `package-lock.json` and `.dockerignore` are the same as in App 2 (only the `name` field differs), so everything in [App 2](#app-2-build-a-more-complicated-nodejs-app) applies here too. The `node` user also exists in the Alpine image, with the same UID (`1000`), so `USER node` and `--chown=node` work without changes.

### What is Alpine Linux?

Every image is built on a Linux distribution. `node:20` is built on **Debian** (version 12, "bookworm"). `node:20-alpine` is built on **Alpine Linux**, a distribution made to be as small and simple as possible. Alpine is built from three small parts:

- **musl** instead of glibc, as the C library: the low-level code that almost every program, including Node.js, uses to talk to the system.
- **BusyBox** instead of the GNU tools: one small program that provides `sh`, `ls`, `ps`, `cp` and most other basic commands.
- **apk** instead of `apt-get`, as the package manager.

The Node.js version is the same in both. In these images, both run Node.js `v20.20.2`.

### Alpine and Debian compared

Measured on this project:

| | `node:20` (Debian) | `node:20-slim` (Debian) | `node:20-alpine` (Alpine) |
| --- | --- | --- | --- |
| Base image size | 1.1 GB | 222 MB | **136 MB** |
| This app's image size | 1.11 GB (App 2) | | **146 MB** (App 3) |
| Installed OS packages | 413 | | 18 |
| C library | glibc | glibc | musl |
| Shell | `bash` (and `sh`) | `bash` (and `sh`) | `sh` only (no `bash`) |
| Package manager | `apt-get` | `apt-get` | `apk` |
| Add a user | `useradd` | `useradd` | `adduser` |
| Extra tools included | `git`, `curl`, `python3`, `make`, `gcc` and more | Almost none, not even `ps` | Almost none (`wget`, BusyBox tools) |
| Memory used by the running app | about 21 MiB | | about 20 MiB |

The app itself, its dependencies and Fastify add only about 10 MB in both cases. The difference is almost all in the base image.

`node:20-slim` is a third option: Debian (glibc, `apt-get`, `bash`), but with most extra packages removed.

### Benefits of Alpine

- **Much smaller images.** 146 MB instead of 1.11 GB, about 8 times smaller. They use less disk space on your machine and on servers.
- **Faster pulls and pushes.** Uploading to a registry and downloading on a server or in CI takes much less time. This matters when you deploy often or start containers on many machines.
- **Smaller attack surface.** 18 packages instead of 413. Fewer programs mean fewer possible security holes, and fewer warnings when you scan the image with a security scanner (for example `docker scout cves`, see [Security scanning: docker scout](#security-scanning-docker-scout)).
- **Fewer tools for an attacker.** If someone breaks into the container, there is no `bash`, `git`, `curl` or compiler to help them.
- **Nothing else changes for this app.** Fastify is plain JavaScript, so it runs on Alpine without changes.

Alpine does **not** make the app faster or use less memory: the running app used about 20 MiB in both images. The benefit is the size of the image, not the speed of the app.

### Things that work differently on Alpine

- **No `bash`.** `docker exec -it fa bash` fails. Use `sh` instead: `docker exec -it fa sh`. If you need bash, install it with `RUN apk add --no-cache bash` (as `root`, before `USER`).
- **No `useradd`.** Use `adduser -D <name>`. See the note in [Create your own user](#create-your-own-user-this-dockerfile).
- **`apk` instead of `apt-get`.** For example, `RUN apk add --no-cache curl` instead of `RUN apt-get update && apt-get install -y curl`. `--no-cache` stops apk from keeping its package list in the image.
- **BusyBox commands have fewer options.** For example, `ps aux` shows only `PID`, `USER`, `TIME` and `COMMAND`. See [ps aux](#processes-inside-a-container-ps-aux).
- **musl instead of glibc.** This only matters for npm packages with **native code** (C or C++ compiled for the system, such as `bcrypt`, `sharp` or some database drivers):
  - If the package ships a ready-made build for musl, it works.
  - If it does not, `npm ci` has to compile it, and Alpine has no compiler. You need `RUN apk add --no-cache python3 make g++` first, which makes the build slower and the image bigger.
  - Some programs exist only for glibc and do not run on Alpine at all, for example the Chrome that Puppeteer downloads.
- **Small behaviour differences.** musl handles some things, like DNS lookups and memory allocation, a little differently from glibc. It rarely matters, but it can cause hard-to-find bugs that only happen on Alpine.

### When to use Alpine, slim or the full image

**Use Alpine (`node:20-alpine`) when:**

- Your dependencies are plain JavaScript, or all have musl builds, like this Fastify app.
- You build images for production, CI or deployments, where size and pull time matter.
- You want the smallest attack surface.

**Use slim (`node:20-slim`) when:**

- You want a small image, but a dependency needs glibc or does not work on Alpine.
- You want Debian's `apt-get` and `bash`, with most of the size savings.

**Use the full image (`node:20`) when:**

- You are learning or debugging and want tools like `git`, `curl`, `bash` and `ps` ready to use.
- `npm ci` needs to compile native modules, and you do not want to install build tools yourself.
- Image size does not matter, for example for a quick local experiment.

A common setup is to start with `node:20` while developing, then switch to `node:20-alpine` (or `node:20-slim` if something breaks) for the image you deploy. Always run and test the app on the same base image you deploy with.

### Build and run the Alpine app

```sh
cd alpine-linux
docker build . -t my-fastify-app:alpine
docker run --init --publish 8080:8080 --rm --name fa my-fastify-app:alpine
```

The tag `alpine` keeps it apart from the Debian build of the same app. Compare the two sizes with:

```sh
docker image ls my-fastify-app
```

The app works exactly like App 2: <http://localhost:8080> returns `{"hello":"world"}`, and the logs are the same JSON lines. For a shell inside the container, use `sh`:

```sh
docker exec -it fa sh
```

**Note:** These commands build whatever `Dockerfile` is in the folder, and that is now Version 2.

### Version 2: install Node.js on plain Alpine

Version 2 does not use a ready-made Node.js image. It starts from plain Alpine Linux and installs Node.js and npm with `apk`:

```dockerfile
FROM alpine:3.19

RUN apk add --update nodejs npm

RUN addgroup -S node && adduser -S node -G node

USER node

WORKDIR /home/node/code

COPY --chown=node package*.json ./

RUN npm ci

COPY --chown=node . .

CMD [ "node", "index.js"]
```

Only the first three instructions are new. From `USER node` down, it is the same as Version 1 and App 2.

#### How the new lines work

- `FROM alpine:3.19` uses the official Alpine Linux image, version 3.19. It is only **7.7 MB** and has no Node.js, npm or `node` user.
- `RUN apk add --update nodejs npm` installs Node.js and npm from Alpine's own package repository:
  - `--update` downloads the latest package list first, so apk knows which versions are available.
  - `nodejs` and `npm` are two separate packages in Alpine, so you need both. Without `npm`, `RUN npm ci` fails.
  - The version is the one Alpine 3.19 ships: Node.js **v20.15.1** and npm **10.2.5**. You cannot pick another patch version.
  - apk installs a few more packages that Node.js needs (for example ICU for languages and dates, and OpenSSL), so the image has 27 packages instead of 18.
- `RUN addgroup -S node && adduser -S node -G node` creates the `node` user, because plain Alpine does not have it:
  - `addgroup -S node` creates a **system** group called `node`.
  - `adduser -S node` creates a **system** user called `node`: it has no password, its login shell is `/sbin/nologin` (nobody can log in as it), and its home folder is `/home/node`. System users get a low UID, here `100`, instead of `1000`.
  - `-G node` puts the user in the `node` group.
  - `&&` runs the second command only if the first one succeeds. Both are in one `RUN`, so they make one layer.

`node` is just a name here. It matches the user in the official images, so the rest of the Dockerfile works without changes.

#### Things that are different in Version 2

- **No `docker-entrypoint.sh`.** Plain Alpine has no `ENTRYPOINT`, so Docker runs the `CMD` directly. `ps aux` shows `/sbin/docker-init -- node index.js` instead of `/sbin/docker-init -- docker-entrypoint.sh node index.js`.
- **Different user IDs.** `id` shows `uid=100(node) gid=101(node)`, not `1000`. This matters when you share files between your machine and the container (for example with `--volume`), because file permissions use the number, not the name.
- **apk's package list stays in the image.** `--update` saves the downloaded list in `/var/cache/apk` (2.1 MB here). Use `--no-cache` instead: it downloads the list, installs and leaves nothing behind:

  ```dockerfile
  RUN apk add --no-cache nodejs npm
  ```

### Version 1 and Version 2 compared

Measured on this project:

| | Version 1: `node:20-alpine` | Version 2: `alpine:3.19` + apk |
| --- | --- | --- |
| Base image size | 136 MB | 7.7 MB (before Node.js is installed) |
| This app's image size | 146 MB | **82.4 MB** |
| Node.js version | v20.20.2 | v20.15.1 |
| npm version | 10.8.2 | 10.2.5 |
| Who builds Node.js | The Node.js Docker team | The Alpine Linux team |
| Choosing the Node.js version | Any version, by tag (`node:20.20.2-alpine`, `node:22-alpine`, ...) | Only the version the Alpine release ships |
| Installed packages | 18 | 27 |
| `node` user | Built in, UID `1000`, shell `/bin/sh` | Created by you, UID `100`, shell `/sbin/nologin` |
| `ENTRYPOINT` | `docker-entrypoint.sh` | None |
| Lines in the Dockerfile | 7 instructions | 9 instructions |
| Memory used by the running app | about 20 MiB | about 20 MiB |

Both versions return `{"hello":"world"}` and log the same JSON lines.

**Why is Version 2 smaller?** The official image includes extra tools such as `yarn`, and Node.js is built to carry its own copies of some libraries. Alpine's `nodejs` package uses the libraries shared by the whole system instead, and has nothing extra.

### Which version to use

**Version 2 benefits:**

- **Smallest image.** 82.4 MB instead of 146 MB, about 44% smaller, and about 13 times smaller than the Debian App 2 (1.11 GB).
- **You control everything.** Every package in the image is one you asked for.
- **The same pattern works for any app.** You can install Python, Go or anything else from apk the same way, on the same base image.

**Version 2 drawbacks:**

- **Older Node.js, chosen by Alpine.** You get the version Alpine 3.19 ships (v20.15.1), not the newest v20 release (v20.20.2). The newer releases include security fixes, and you cannot upgrade without changing the Alpine version.
- **Alpine 3.19 is past its end of support.** Its official support ended in November 2025. Some fixes still reached the 3.19 package repository after that (for example a 2026 fix for musl), but they are no longer promised, and the newest 3.19 image (3.19.9) still has 4 known vulnerabilities. See [Security scanning: docker scout](#security-scanning-docker-scout). Use a supported Alpine release, and check which Node.js version its `nodejs` package has at [pkgs.alpinelinux.org](https://pkgs.alpinelinux.org/packages?name=nodejs).
- **More to maintain.** You write and look after the install and user lines that the official image already does for you.
- **Different from the official image.** The UID (`100`, not `1000`) and the missing `ENTRYPOINT` can surprise people who know the official `node` images.

**Use Version 1 (`node:20-alpine`) when** you want an exact Node.js version, regular security updates from the Node.js team and the least work. This is the right choice for most projects.

**Use Version 2 (`alpine` + `apk`) when** image size matters most, the Node.js version from Alpine is good enough for you, and you keep the Alpine version up to date. It is also a good way to learn what the official image does for you.

Build Version 2 with its own tag, so you can compare it with Version 1:

```sh
cd alpine-linux
docker build . -t my-fastify-app:alpine-apk
docker image ls my-fastify-app
```

---

## App 4: multi-stage-builds

This app, in the `multi-stage-builds/` folder, runs the same Fastify app as App 2 and App 3. `index.js`, `package.json`, `package-lock.json` and `.dockerignore` are the same (only the `name` field differs). The difference is the `Dockerfile`: it builds the app in **two stages**.

```dockerfile
# build step
FROM node:20 AS node-builder
RUN mkdir /build
WORKDIR /build
COPY package*.json ./
RUN npm ci
COPY . .

#production step
FROM alpine:3.19
RUN apk add --update nodejs npm
RUN addgroup -S node && adduser -S node -G node
USER node
RUN mkdir /home/node/code
WORKDIR /home/node/code
COPY --from=node-builder --chown=node:node /build .
CMD ["node", "index.js"]
```

### What is a multi-stage build?

A Dockerfile can have more than one `FROM`. Each `FROM` starts a new **stage**, with its own base image and its own layers. A later stage can copy files from an earlier one with `COPY --from=<stage>`.

Only the **last** stage becomes the image. Everything in the earlier stages (their base image, tools and temporary files) is thrown away, except the files you copy out.

This lets you split the work in two:

1. **Build stage:** a big image with all the tools you need to build the app. Here, `node:20` (Debian, 1.1 GB) runs `npm ci`.
2. **Production stage:** a small image with only what the app needs to **run**. Here, plain `alpine:3.19` with Node.js.

Think of the build stage as a workshop and the production stage as the box you ship: you build in the workshop, then put only the finished product in the box.

### How this Dockerfile works

#### Stage 1: build (`node-builder`)

- `FROM node:20 AS node-builder` starts the first stage from the full Debian Node.js image. `AS node-builder` gives the stage a name, so later lines can refer to it.
- `RUN mkdir /build` creates the folder `/build`. (`WORKDIR` on the next line would create it anyway, so this line is not needed.)
- `WORKDIR /build` makes `/build` the current directory.
- `COPY package*.json ./` and `RUN npm ci` install the dependencies into `/build/node_modules`, in the same cache-friendly order as App 2. See [Faster builds: copy package files first](#faster-builds-copy-package-files-first).
- `COPY . .` copies the rest of the app.

At the end of this stage, `/build` holds the complete, ready-to-run app: `index.js`, `package.json`, `package-lock.json` and `node_modules`.

This stage has no `USER`, so it runs as `root`. That is fine here, because nothing from this stage runs in the final container. Only its files are copied out.

#### Stage 2: production

- `FROM alpine:3.19` starts a new, empty stage from plain Alpine. Nothing from stage 1 is here yet.
- `RUN apk add --update nodejs npm`, `RUN addgroup -S node && adduser -S node -G node` and `USER node` are the same as [App 3 Version 2](#version-2-install-nodejs-on-plain-alpine).
- `RUN mkdir /home/node/code` creates the app folder as `node`. (`WORKDIR` would create it anyway, so this line is not needed either.)
- `WORKDIR /home/node/code` makes it the current directory.
- `COPY --from=node-builder --chown=node:node /build .` is the key line:
  - `--from=node-builder` copies from the `node-builder` stage, **not** from your project folder.
  - `/build` is the folder to copy from in that stage, and `.` is the current directory here (`/home/node/code`).
  - `--chown=node:node` makes `node` the owner of every copied file, including `node_modules`.
- `CMD ["node", "index.js"]` starts the server.

The production stage never runs `npm ci`. It receives `node_modules` already installed.

### What ends up in the final image

`docker history` on the final image shows only the production stage's layers:

| Layer | Size |
| --- | --- |
| `alpine:3.19` base | 7.7 MB |
| `RUN apk add --update nodejs npm` | 64.6 MB |
| `RUN addgroup ... adduser ...` | 4.7 kB |
| `RUN mkdir`, `USER`, `WORKDIR`, `CMD` | 0 B |
| `COPY --from=node-builder ... /build .` | 8.35 MB |
| **Total** | **80.7 MB** |

The 1.1 GB `node:20` image, the npm download cache and everything else in stage 1 are **not** in it. Inside the container, `/home/node/code` has the same files as in App 3, all owned by `node`.

### Build and run the multi-stage app

```sh
cd multi-stage-builds
docker build . -t my-fastify-app:multi-stage
docker run --init --publish 8080:8080 --rm --name fa my-fastify-app:multi-stage
```

The build output names each step with its stage, for example `[node-builder 5/6] RUN npm ci` and `[stage-1 6/6] COPY --from=node-builder ...`. The production stage has no `AS` name, so Docker calls it `stage-1` (stages are numbered from 0). BuildKit runs steps from different stages **at the same time** when they do not depend on each other. For example, `apk add` in stage 2 runs while `npm ci` runs in stage 1.

The app works like App 2 and App 3: <http://localhost:8080> returns `{"hello":"world"}`, and `ps aux` shows `/sbin/docker-init -- node index.js`.

To build **only** the first stage, for example to debug it, use `--target`:

```sh
docker build . --target node-builder -t my-fastify-app:builder   # 1.11 GB
```

### Single-stage and multi-stage compared

Measured on this project, with the same app:

| | App 2 | App 3 (Version 2) | App 4 |
| --- | --- | --- | --- |
| Dockerfile | One stage, `node:20` | One stage, `alpine:3.19` + apk | Build on `node:20`, run on `alpine:3.19` |
| Where `npm ci` runs | In the final image | In the final image | In the build stage only |
| Image size | 1.11 GB | 82.4 MB | **80.7 MB** |
| Node.js version that runs the app | v20.20.2 | v20.15.1 | v20.15.1 |

App 4 is only slightly smaller than App 3 (Version 2), because both end with Alpine plus Node.js and npm. The difference is files that `npm ci` leaves behind, like its download cache, which stay in stage 1. The big win of multi-stage builds comes when the build needs heavy tools the app does not need to run, such as compilers, TypeScript or a frontend bundler. See [When to use multi-stage builds](#when-to-use-multi-stage-builds).

### Watch out: native modules

In this Dockerfile, `npm ci` runs on **Debian** (glibc, Node.js v20.20.2), but the app runs on **Alpine** (musl, Node.js v20.15.1). See [Things that work differently on Alpine](#things-that-work-differently-on-alpine).

- **Fastify is plain JavaScript**, so the copied `node_modules` works on Alpine without problems.
- **A package with native code** (such as `bcrypt` or `sharp`) would be compiled or downloaded for Debian and glibc in stage 1. Copied to Alpine, it would fail to load, with errors about missing libraries or symbols.

To be safe, build and run on the **same** system and Node.js version, for example both on Alpine:

```dockerfile
FROM node:20-alpine AS node-builder
...

FROM node:20-alpine
...
```

### Making it smaller and simpler

These changes are optional. They are tested on this project:

- **Do not install npm in the production stage.** The app is already installed when it is copied in, so only `node` is needed to run it. `--no-cache` also leaves out apk's package list:

  ```dockerfile
  RUN apk add --no-cache nodejs
  ```

  The image shrinks from **80.7 MB** to **71.1 MB**, and the app still runs. Without npm in the image, nobody can install packages into the running container.
- **Remove the two `RUN mkdir` lines.** `WORKDIR` creates the folder anyway (owned by the current `USER`, see [More about WORKDIR](#more-about-workdir)), and each `RUN` adds a step to the build.
- **Name the production stage**, for example `FROM alpine:3.19 AS production`. Then the build output says `production` instead of `stage-1`.
- **Use a supported Alpine version.** Alpine 3.19 is past its end of support, and its fixes are no longer promised. See [Which version to use](#which-version-to-use) and [docker scout](#security-scanning-docker-scout).

### When to use multi-stage builds

**Benefits:**

- **Smaller final image.** Build tools, caches and source files stay in the build stage.
- **More secure.** Compilers, package managers and build secrets are not in the image you run, so there is less to attack.
- **One Dockerfile for building and running.** You do not need separate build scripts on your machine.
- **Build stages can use any image.** For example, build with a full `node:20` and run on a small `alpine`.

**Use a multi-stage build when:**

- The build needs tools the app does not need to run: TypeScript (`tsc`), a bundler (Vite, webpack), compilers for native modules, or dev dependencies for tests.
- The build creates output that is much smaller than what was needed to make it, for example a frontend built into static files, or a Go or Rust program compiled into one binary.
- You want the production image to be as small and safe as possible.

**A single stage is enough when** the app needs nothing at build time except `npm ci`, like this Fastify app. Then a small single-stage image gets nearly the same result with fewer lines and no risk of a system mismatch: [App 3 Version 2](#version-2-install-nodejs-on-plain-alpine) is 82.4 MB, against 80.7 MB here. [App 3 Version 1](#app-3-alpine-linux) (`node:20-alpine`, 146 MB) is bigger, but gives you an up-to-date Node.js.

---

## App 5: distroless

This app, in the `distroless/` folder, is the same Fastify app again, with a multi-stage build like App 4. The difference is the production stage: instead of Alpine, it runs on a **distroless** image.

```dockerfile
# build stage
FROM node:20 AS node-builder
WORKDIR /build
COPY package-lock.json package.json ./
RUN npm ci
COPY . .

# runtime stage
FROM gcr.io/distroless/nodejs20
COPY --from=node-builder --chown=node:node /build /app
WORKDIR /app
CMD ["index.js"]
```

### What is distroless?

[Distroless](https://github.com/GoogleContainerTools/distroless) images are made by Google. They contain **only your app's runtime** (here Node.js) and the few system files it needs, such as certificates, time zone data and `/etc/passwd`. They have **no** Linux distribution tools at all:

- no shell (`sh` or `bash`)
- no package manager (`apt-get` or `apk`)
- no basic commands (`ls`, `cat`, `ps`, `whoami`)

They are built on Debian 12, so they use **glibc**, like `node:20`, not musl like Alpine.

| | Alpine (App 4's production stage) | Distroless (App 5's production stage) |
| --- | --- | --- |
| Based on | Alpine Linux | Debian 12, with almost everything removed |
| C library | musl | glibc |
| Shell and basic commands | `sh` and BusyBox | None |
| Package manager | `apk` | None |
| Node.js | Installed by you with `apk` (v20.15.1) | Built into the image (v20.20.2) |

The idea: a container only needs to **run one program**. Everything else in the image is something an attacker could use, and something that can have security holes.

### How the distroless Dockerfile works

#### Stage 1: build (`node-builder`)

The same as App 4, with two small differences:

- There is no `RUN mkdir /build`. `WORKDIR /build` creates the folder by itself.
- `COPY package-lock.json package.json ./` names both files instead of using `package*.json`. The result is the same.

#### Stage 2: runtime

- `FROM gcr.io/distroless/nodejs20` uses the distroless Node.js 20 image. It comes from Google's registry (`gcr.io`), not Docker Hub, so the image name includes the registry address. Without a tag, Docker uses `latest`.
- `COPY --from=node-builder --chown=node:node /build /app` copies the installed app from stage 1 into `/app`. Docker creates `/app`. (`--chown=node:node` does not work here, see [Problem 1: the app runs as root](#problem-1-the-app-runs-as-root).)
- `WORKDIR /app` makes `/app` the current directory.
- `CMD ["index.js"]` gives only the file name, **not** `node index.js`. The distroless image already sets `ENTRYPOINT ["/nodejs/bin/node"]`, and Docker adds the `CMD` to the end of the `ENTRYPOINT`. The container runs `/nodejs/bin/node index.js`.

There is no `RUN` in this stage, and there could not be: `RUN` needs a shell, and distroless has none. Everything that needs commands must happen in the build stage.

#### ENTRYPOINT and CMD

This is the first app in this README where the difference matters:

| | What it is | Set by |
| --- | --- | --- |
| `ENTRYPOINT` | The program that always runs. | The distroless image: `/nodejs/bin/node` |
| `CMD` | The default **arguments** for the `ENTRYPOINT`. You can replace them in `docker run`. | This Dockerfile: `index.js` |

So `docker run more-complicated-app:distroless other.js` runs `node other.js`. If you wrote `CMD ["node", "index.js"]` here, the container would run `node node index.js` and fail, because there is no file called `node`. The `node` images use the `ENTRYPOINT` `docker-entrypoint.sh` for the same reason (see [ps aux](#processes-inside-a-container-ps-aux)).

### Build and run the distroless app

```sh
cd distroless
docker build . -t more-complicated-app:distroless
docker run --init --publish 8080:8080 --rm --name dl more-complicated-app:distroless
```

The app works like the others: <http://localhost:8080> returns `{"hello":"world"}`, with the same JSON logs. `--init` works too, because Docker adds `docker-init` from the outside, so it does not need a shell in the image.

The image is **145 MB**: about 137 MB for the distroless Node.js image and 8.35 MB for the app.

`docker image ls` shows the distroless base image as created "56 years ago". This is not a mistake: distroless images are built with a fixed date (1 January 1970), so that building the same image twice gives exactly the same result.

### Living without a shell

With no shell, the usual ways to look inside a container fail:

```sh
docker exec -it dl sh
# exec: "sh": executable file not found in $PATH
```

What still works:

- **`docker logs dl`** shows the app's output.
- **`docker top dl`** lists the processes from outside the container, so it does not need `ps`.
- **Node.js itself** is in the image, so you can run JavaScript with it:

  ```sh
  docker exec dl /nodejs/bin/node -e 'console.log(process.getuid())'
  ```

- **The `:debug` images** add a BusyBox shell, for debugging only. Do not deploy them:

  ```sh
  docker run -it --rm --entrypoint sh gcr.io/distroless/nodejs20-debian12:debug
  ```

### Problem 1: the app runs as root

This Dockerfile has **no `USER`**, so the app runs as `root`. `docker top dl` shows `root` in the `USER` column, and `process.getuid()` prints `0`. This undoes the work from [Running as a non-root user](#running-as-a-non-root-user).

`--chown=node:node` does not help, because **distroless has no `node` user**. Its `/etc/passwd` has only three users:

| User | UID | Purpose |
| --- | --- | --- |
| `root` | `0` | The default user. |
| `nobody` | `65534` | A user with no rights. |
| `nonroot` | `65532` | The normal user for running apps. |

Docker did not stop with an error. The files simply ended up owned by `root` (`0:0`). You cannot create a `node` user either, because `adduser` needs a shell.

**The fix:** use the `:nonroot` image, which already runs as `nonroot`, and give the files to that user:

```dockerfile
FROM gcr.io/distroless/nodejs20-debian12:nonroot
COPY --from=node-builder --chown=nonroot:nonroot /build /app
WORKDIR /app
CMD ["index.js"]
```

Tested on this project: the app runs as UID `65532`, the files are owned by `65532`, and it still returns `{"hello":"world"}`. `nodejs20-debian12` is the full name of the same image, with the Debian version written out.

### Problem 2: no .dockerignore

The other apps have a `.dockerignore`, but this folder does not. So `COPY . .` in stage 1 copies **everything** in the folder:

- Your local `node_modules` (installed on macOS) is copied **over** the one that `npm ci` just installed for Linux. This app still works, because Fastify is plain JavaScript. But any package with native code would break. See [Leaving files out: .dockerignore](#leaving-files-out-dockerignore).
- The build sends 8.5 MB to Docker instead of a few kB, so every build starts slower.
- The `Dockerfile` itself ends up in `/app`.

**The fix:** add the same `.dockerignore` as the other apps:

```
node_modules/
.git/
```

### Distroless compared with the other apps

Measured on this project, with the same Fastify app:

| | App 2 | App 4 | App 5 |
| --- | --- | --- | --- |
| Runs on | `node:20` (Debian) | `alpine:3.19` + apk | `gcr.io/distroless/nodejs20` |
| Image size | 1.11 GB | 80.7 MB | 145 MB |
| Node.js version | v20.20.2 | v20.15.1 | v20.20.2 |
| Built and run on the same system | Yes | No (Debian, then Alpine) | Yes (both Debian 12, glibc) |
| Shell | `bash` | `sh` | None |
| Package manager | `apt-get` | `apk` | None |
| User | `node` | `node` (UID 100) | `root` (or `nonroot` with `:nonroot`) |

- **Distroless is bigger than Alpine** (145 MB against 80.7 MB), mostly because its Node.js layer (99 MB) is the full official build, which is bigger than Alpine's `nodejs` package. But it has far fewer **programs**: only Node.js, with nothing an attacker can use to explore or change the container.
- **No native-module problem.** Stage 1 (`node:20`) and stage 2 are both Debian 12 with glibc, and both run Node.js v20.20.2. Packages with native code built in stage 1 work in stage 2, unlike in [App 4](#watch-out-native-modules).
- **Up-to-date Node.js.** The image comes with Node.js already built in, and Google keeps it updated. You do not depend on the version an Alpine release ships.

### When to use distroless

**Benefits:**

- **Very small attack surface.** No shell, no package manager and no basic commands, so an attacker who gets in has almost nothing to work with.
- **Fewer security warnings.** Security scanners find far fewer packages to report on.
- **glibc, like the build stage.** Native modules built on `node:20` work without changes.
- **Current Node.js**, maintained by Google.

**Drawbacks:**

- **Hard to debug.** No shell, no `ls`, no `ps`. You need `docker logs`, `docker top` or a `:debug` image.
- **Must be a multi-stage build.** You cannot run `npm ci` or any other command in the runtime image.
- **No `node` user, and root by default.** Use the `:nonroot` image.
- **Fewer choices.** Google publishes only some runtimes and versions. You cannot add a package if the app needs one.

**Use distroless when** the app is going to production, you already use a multi-stage build, and security matters more than easy debugging. Use the `:nonroot` image.

**Use Alpine or `node:20-slim` when** you need a shell to debug or run scripts in the container, or the app needs extra system packages at run time.

---

## App 6: static-asset-project

Apps 1 to 5 were Node.js **servers**: Node.js had to run inside the container to answer each request. This app is a **static website**. Node.js is needed only to **build** it into plain HTML, CSS and JavaScript files. After that, any web server can send those files, and Node.js is not needed at all.

```dockerfile
# build step
FROM node:20 AS node-builder
WORKDIR /app
COPY . .
RUN npm ci
RUN npm run build

#production step
FROM nginx:alpine
COPY --from=node-builder /app/dist /usr/share/nginx/html
```

This is the multi-stage idea from [App 4](#app-4-multi-stage-builds) taken all the way: the final image has **no Node.js**, only a web server and the built files.

### The app: an Astro website

The folder is the "basics" starter project of [Astro](https://astro.build), a framework for building websites:

- `src/pages/index.astro` is the home page. `src/layouts/` and `src/components/` hold the parts it is built from.
- `public/` holds files that are copied as they are, like `favicon.svg`.
- `astro.config.mjs` turns on React and [Tailwind CSS](https://tailwindcss.com) (a CSS framework). `tailwind.config.mjs` and `tsconfig.json` configure Tailwind and TypeScript.
- `package.json` lists Astro, React, Tailwind and TypeScript, with about 400 packages in `node_modules` (198 MB).

The `build` script in `package.json` is:

```json
"build": "astro check && astro build"
```

- `astro check` checks the code for TypeScript errors. The build stops if it finds any.
- `astro build` turns the project into a static website in the `dist/` folder.

The result is small: `dist/` is only **155 kB** (`index.html`, `favicon.svg` and an `_astro/` folder with the CSS and JavaScript). That is what goes into the final image.

### How the static site Dockerfile works

#### Stage 1: build (`node-builder`)

- `FROM node:20 AS node-builder` uses the full Debian Node.js image, which has everything the build needs.
- `WORKDIR /app` creates `/app` and makes it the current directory.
- `COPY . .` copies the whole project folder. See [Improving the static site build](#improving-the-static-site-build) for two problems with copying everything at this point.
- `RUN npm ci` installs the dependencies.
- `RUN npm run build` runs the `build` script, which creates `/app/dist`.

This stage has no `USER` and runs as `root`. As in App 5, that is fine, because none of it runs in the final container.

#### Stage 2: production

- `FROM nginx:alpine` uses the official [nginx](#what-is-nginx) image, built on Alpine.
- `COPY --from=node-builder /app/dist /usr/share/nginx/html` copies the built website into the folder that nginx serves files from.

There is **no `CMD`**. The `nginx:alpine` image already has one (`nginx -g daemon off;`), and this stage keeps it. It also keeps the image's `ENTRYPOINT` (`/docker-entrypoint.sh`) and its `EXPOSE 80`.

There is no `--chown` either. nginx only **reads** these files, and they are readable by everyone, so `root` can own them.

### What is nginx?

[nginx](https://nginx.org) ("engine x") is a web server. It is very fast at sending files, and it is often also used in front of other apps (as a reverse proxy or load balancer).

The `nginx` image is set up to work without any configuration:

| Setting | Value |
| --- | --- |
| Folder it serves files from | `/usr/share/nginx/html` |
| Port it listens on | `80` (the image declares `EXPOSE 80`) |
| Default file for a folder | `index.html`, so `/` returns `/usr/share/nginx/html/index.html` |
| Command | `nginx -g daemon off;` |

`daemon off;` keeps nginx in the foreground. Normally nginx starts and moves itself into the background, but in a container the main process must keep running, or the container stops (see [Earlier version: a one-line script](#earlier-version-a-one-line-script)).

### Build and run the static site

```sh
cd static-asset-project
docker build . -t my-static-site
docker run --publish 8080:80 --rm --name site my-static-site
```

- `--publish 8080:80` maps port `8080` on your machine to nginx's port `80` in the container. Open <http://localhost:8080> to see the "Welcome to Astro." page.
- `--init` is not needed. nginx is made to run as PID 1 and handles `Ctrl+C` and `docker stop` itself, so `docker stop site` stops it in about a second.

The build takes about 25 seconds the first time: `npm ci` takes about 12 seconds, and `npm run build` about 6. The build output shows `astro check` reporting `0 errors`, then `astro build` building the pages.

nginx logs one line per request, in the standard web server format:

```
172.17.0.1 - - [29/Oct/2026:18:30:23 +0000] "GET /favicon.svg HTTP/1.1" 200 749 "-" "curl/8.7.1" "-"
```

A file that does not exist returns `404` and also logs an `[error]` line.

### What is inside the final image

| | |
| --- | --- |
| Image size | **62.8 MB**, of which the website is 155 kB |
| Build stage size (thrown away) | 1.51 GB |
| Node.js | Not installed |
| nginx version | 1.31.8 |

`ls -la /usr/share/nginx/html` in the container shows the built site, plus `50x.html`, the error page that comes with nginx:

```
50x.html
_astro/
favicon.svg
index.html
```

`docker exec site ps aux` shows how nginx runs:

```
PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process nginx -g daemon off;
   30 nginx     0:00 nginx: worker process
   31 nginx     0:00 nginx: worker process
   ...
```

- The **master** process runs as `root`. It reads the configuration, opens port 80 and starts the workers.
- The **worker** processes answer the requests. They run as the `nginx` user, not `root`. There is one per CPU core.

So nginx does its own [non-root](#running-as-a-non-root-user) setup: the processes that handle requests from the internet do not run as `root`, even without a `USER` line.

### Your Dockerfile and solution.Dockerfile

The folder also has `solution.Dockerfile`. Its build stage is the same. The only difference is the production base image:

```dockerfile
# you could totally use nginx:alpine here too
FROM nginx:latest
```

| | `Dockerfile` (`nginx:alpine`) | `solution.Dockerfile` (`nginx:latest`) |
| --- | --- | --- |
| Base | Alpine | Debian 13 |
| nginx version | 1.31.8 | 1.31.8 |
| Image size | **62.8 MB** | 183 MB |

Both serve the same website with the same nginx version. The Alpine version is about 3 times smaller. nginx does not need any native npm modules, so Alpine's [musl differences](#things-that-work-differently-on-alpine) do not matter here.

To build the other file, give its name with `-f`:

```sh
docker build . -f solution.Dockerfile -t my-static-site:debian
```

### Serve the site with a bind mount

You do not have to build an image to try the website. If you have built the site on your machine (`npm run build`, which creates `dist/`), you can **lend** that folder to a plain nginx container:

```sh
cd static-asset-project
npm run build
docker run --mount type=bind,source="$(pwd)"/dist,target=/usr/share/nginx/html -p 8080:80 nginx:latest
```

Open <http://localhost:8080> and you see the same "Welcome to Astro." page. No Dockerfile and no `docker build` are needed: the container uses the official `nginx:latest` image unchanged, and reads the files straight from your `dist/` folder.

| Part | What it does |
| --- | --- |
| `docker run ... nginx:latest` | Starts a container from the official nginx image (Debian). |
| `--mount` | Attaches storage to the container. |
| `type=bind` | The storage is a **bind mount**: a folder from your machine. See [Bind mounts](#bind-mounts). |
| `source="$(pwd)"/dist` | The folder on **your machine**. `$(pwd)` is replaced by the current folder, so this becomes `/Users/.../static-asset-project/dist`. The quotes keep it working if the path has spaces. |
| `target=/usr/share/nginx/html` | Where the folder appears **inside the container**: the folder nginx serves files from. |
| `-p 8080:80` | Maps port `8080` on your machine to nginx's port `80`. |

The command runs in the foreground. Press `Ctrl+C` to stop it. It has no `--rm`, so the stopped container stays in `docker ps --all` until you remove it. Add `--rm` (and `--name site`) to clean up automatically.

#### Why use a bind mount here?

- **Changes show up at once.** Run `npm run build` again, and refresh the browser: nginx serves the new files right away. You do not rebuild the image or restart the container.
- **Quick testing.** You can check how the built site looks in real nginx in a few seconds.

#### What happens to the files in the image?

The bind mount **hides** whatever the image had in `/usr/share/nginx/html`. The image's own `index.html` and `50x.html` are still there underneath, but the container only sees your `dist/` folder. Here `/50x.html` returns `404`, because `dist/` does not have it.

Inside the container:

```sh
docker exec <container> ls -la /usr/share/nginx/html
# _astro  favicon.svg  index.html   (your dist/ folder, no 50x.html)
```

`docker inspect` shows the mount:

```json
[{"Type":"bind","Source":"/Users/<you>/.../static-asset-project/dist","Destination":"/usr/share/nginx/html","Mode":"","RW":true,"Propagation":"rprivate"}]
```

`"RW":true` means the container can **write** to your folder. See [Read-only bind mounts](#read-only-bind-mounts).

#### Bind mount or COPY?

| | Bind mount (this command) | `COPY` in a Dockerfile (App 6) |
| --- | --- | --- |
| Where the files are | On your machine, lent to the container | Inside the image |
| Needs `docker build` | No | Yes |
| After you change a file | Visible at once | Rebuild the image |
| Works on another computer | No, the folder must exist on that machine | Yes, the image has everything |
| Good for | Development and quick testing | Production and sharing |

Use a bind mount while you work on the site, and the Dockerfile from App 6 when you ship it.

### Bind mounts

A **bind mount** connects a folder (or a single file) on your machine to a path inside a container. It is not a copy: both sides see **the same files**.

- When you change a file on your machine, the container sees the change at once.
- When the container writes a file, it appears on your machine.
- When the container is deleted, the files stay on your machine.

Tested on this project: after a bind mount, editing `index.html` on the Mac changed the page in the browser without a restart, and a file written by `docker exec` inside the container appeared in the Mac folder and was still there after the container stopped.

#### Two ways to write it: --mount and -v

```sh
# --mount: longer, every part is named
docker run --mount type=bind,source="$(pwd)"/dist,target=/usr/share/nginx/html -p 8080:80 nginx:latest

# -v (--volume): shorter, SOURCE:TARGET
docker run -v "$(pwd)"/dist:/usr/share/nginx/html -p 8080:80 nginx:latest
```

Both do the same thing, with one important difference, tested here:

| If the source folder does not exist | |
| --- | --- |
| `--mount` | Fails with `bind source path does not exist`. |
| `-v` | Creates an **empty** folder and mounts it. nginx then serves an empty folder, so you get `403 Forbidden` instead of your site, with no warning about the path. |

`--mount` is the clearer and safer choice, because a typo in the path gives an error instead of a silent empty folder.

#### The source must be an absolute path

```sh
docker run --mount type=bind,source=dist,target=/usr/share/nginx/html nginx:latest
# invalid mount path: 'dist' mount path must be absolute
```

That is why the command uses `"$(pwd)"/dist`: it turns the relative `dist` into a full path.

#### Read-only bind mounts

By default, the container can change and delete your files. nginx only needs to read them, so add `readonly` (or `ro`):

```sh
docker run --mount type=bind,source="$(pwd)"/dist,target=/usr/share/nginx/html,readonly -p 8080:80 nginx:latest
```

Tested here: the site still works, and writing from inside the container fails with `Read-only file system`. With `-v`, add `:ro` at the end: `-v "$(pwd)"/dist:/usr/share/nginx/html:ro`.

#### Things to know about bind mounts

- **The container can damage your files.** It runs as `root` here, so without `readonly` it can change or delete anything in the folder. Mount only the folder you need, never your home folder.
- **They depend on the machine.** The path must exist on the computer that runs the container, so a bind mount is not part of the image and does not work on a server unless the folder is there too.
- **File owners can look different.** On Docker Desktop for Mac, files from the Mac can appear inside the container with owners like `root`, and files the container writes appear on the Mac as owned by you.
- **On Mac and Windows they are slower.** Docker runs Linux in a small virtual machine, and every file access is passed between your system and that machine. For folders with many small files, like `node_modules`, this can be noticeably slow.
- **They are not volumes.** A **volume** (`--mount type=volume,source=mydata,target=...`) is storage that Docker creates and manages itself, not a folder you choose. Volumes are better for data that only the container needs, like a database's files. Bind mounts are better when **you** need to see or edit the files, like source code or a built website. See [Volumes](#volumes).

### Improving the static site build

The Dockerfile works, but it has the same two problems that [App 2](#app-2-build-a-more-complicated-nodejs-app) solved:

1. **No `.dockerignore`.** `COPY . .` copies the whole folder, including your local `node_modules` (198 MB), and Docker has to send all of it to the build. The build still works, because `npm ci` deletes `node_modules` first and installs it again, but sending it is slow. Old `dist/` and `.astro/` folders are copied too.
2. **`COPY . .` comes before `RUN npm ci`.** When you change any file, even one line of `index.astro`, the `COPY . .` step changes, so `npm ci` runs again and downloads about 400 packages. See [Faster builds: copy package files first](#faster-builds-copy-package-files-first).

Add a `.dockerignore` (the same folders as the project's `.gitignore`):

```
node_modules/
dist/
.astro/
.git/
```

And copy the package files first:

```dockerfile
FROM node:20 AS node-builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=node-builder /app/dist /usr/share/nginx/html
```

With these two changes, Docker no longer sends `node_modules` to the build, and when only the website's code changes, `npm ci` stays `CACHED`.

### When to serve static files with nginx

**This setup works for** any project that builds to plain files: Astro, Vite, React (Create React App), Vue, Angular, Svelte, Hugo, a documentation site, or hand-written HTML. If the build output is a folder of HTML, CSS and JavaScript, you can serve it with nginx.

**Benefits:**

- **Small image.** 62.8 MB, with no Node.js and no `node_modules`.
- **Fast.** nginx is built to send files, and it does it faster and with less memory than a Node.js server.
- **Safer.** No Node.js, npm or app code runs in the container, so there is less that can have security holes. The workers do not run as `root`.
- **No server code.** You do not write or maintain a server like `index.js` in Apps 1 to 5.

**It does not work when** the site needs code on the server for each request, for example a login, a database, an API, or pages rendered on the server (Astro's server mode, Next.js server features). Then you need a Node.js server, as in Apps 1 to 5. nginx is often still used **in front of** it.

To change how nginx behaves (for example, single-page apps that need every path to return `index.html`, caching headers, or redirects), copy your own configuration file into `/etc/nginx/conf.d/default.conf`.

---

## App 7: docker-volume

All the apps so far kept nothing: when a container was deleted, everything it had written was gone. This app, in the `docker-volume/` folder, shows how to keep data **between containers** with a **volume**.

### The app: a counter that saves to a file

```js
const fs = require("fs").promises;
const path = require("path");

const dataPath = path.join(process.env.DATA_PATH || "./data.txt");

fs.readFile(dataPath)
  .then((buffer) => {
    const data = buffer.toString();
    const dataToWrite = +data + 1;
    console.log(`${dataPath} found, incrementing by 1:`, dataToWrite);
    writeTo(dataToWrite);
  })
  .catch((e) => {
    console.log(`file not found, writing "0" to ${dataPath}`);
    writeTo(0);
  });

const writeTo = (data) => {
  fs.writeFile(dataPath, data.toString()).catch(console.error);
};
```

- It reads the path of its data file from the environment variable `DATA_PATH`. If it is not set, it uses `./data.txt`.
- If the file exists, it reads the number in it, adds 1 and writes it back (`+data` turns the text into a number).
- If the file does not exist, it writes `0`.
- Then it exits. It is not a server, so the container stops after each run.

Each run adds 1, **if** the file survives from one run to the next. That is what the volume is for.

### The incrementor Dockerfile

```dockerfile
FROM node:20-alpine
COPY --chown=node:node . /src
WORKDIR /src
CMD ["node", "index.js"]
```

- `FROM node:20-alpine`: the small Alpine Node.js image. See [App 3](#app-3-alpine-linux).
- `COPY --chown=node:node . /src`: copies the folder (here only `Dockerfile` and `index.js`) to `/src`, owned by `node`.
- `WORKDIR /src`: makes `/src` the current directory, so `node index.js` finds the file.
- `CMD ["node", "index.js"]`: runs the script.

There is **no `USER`**, so the script runs as `root` (`whoami` prints `root`). This matters for volumes: see [Volumes and non-root users](#volumes-and-non-root-users).

There is also no `/data` folder in the image. The volume creates it.

Build it:

```sh
cd docker-volume
docker build . -t incrementor
```

### Run the incrementor with a volume

```sh
docker run --rm --env DATA_PATH=/data/num.txt --mount type=volume,src=incrementor-data,target=/data incrementor
```

| Part | What it does |
| --- | --- |
| `docker run --rm` | Starts a new container and deletes it when it stops. The **volume is not deleted**: `--rm` only removes the container. |
| `--env DATA_PATH=/data/num.txt` | Sets the environment variable `DATA_PATH`, so the script saves to `/data/num.txt`. See [Environment variables](#environment-variables---env). |
| `--mount` | Attaches storage to the container. |
| `type=volume` | The storage is a **volume**, managed by Docker. See [Volumes](#volumes). |
| `src=incrementor-data` | The **name** of the volume. If no volume with this name exists, Docker creates it. `src` is short for `source`. |
| `target=/data` | Where the volume appears inside the container. Files written to `/data` go into the volume. |
| `incrementor` | The image to run. |

Run it a few times. Measured on this project:

```
$ docker run --rm --env DATA_PATH=/data/num.txt --mount type=volume,src=incrementor-data,target=/data incrementor
file not found, writing "0" to /data/num.txt
$ docker run --rm --env DATA_PATH=/data/num.txt --mount type=volume,src=incrementor-data,target=/data incrementor
/data/num.txt found, incrementing by 1: 1
$ docker run --rm --env DATA_PATH=/data/num.txt --mount type=volume,src=incrementor-data,target=/data incrementor
/data/num.txt found, incrementing by 1: 2
```

Every run is a **new** container, and each one is deleted at the end. But the number keeps growing, because `/data/num.txt` lives in the volume, not in the container.

#### Without the volume

Leave out `--mount`, and every run starts from nothing. Here it is even worse: `/data` does not exist in the image, so the script cannot save at all:

```
file not found, writing "0" to /data/num.txt
Error: ENOENT: no such file or directory, open '/data/num.txt'
```

#### Without DATA_PATH

Leave out `--env`, and the script writes `./data.txt`, which is `/src/data.txt` in the container. That is **outside** the volume, so it is lost when the container is deleted. Every run prints `file not found, writing "0" to data.txt`, even though the volume is attached.

The volume only keeps what is written **inside** its `target` folder.

#### The short form: -v

```sh
docker run --rm -e DATA_PATH=/data/num.txt -v incrementor-data:/data incrementor
```

`-e` is `--env`, and `-v NAME:PATH` mounts a volume. When the part before `:` is a **name** (not a path starting with `/`), `-v` creates a volume. When it is a path, `-v` makes a [bind mount](#bind-mounts).

### Environment variables: --env

An **environment variable** is a named setting that a program reads when it starts. In Node.js, they are in `process.env`, so `--env DATA_PATH=/data/num.txt` makes `process.env.DATA_PATH` equal `"/data/num.txt"`.

They let you change how a container behaves **without rebuilding the image**: the same `incrementor` image can save anywhere you choose.

```sh
docker run --env KEY=value ...          # one variable
docker run -e KEY1=a -e KEY2=b ...      # several
docker run --env-file .env ...          # read KEY=value lines from a file
```

A Dockerfile can also set a default with `ENV DATA_PATH=/data/num.txt`, and `--env` overrides it.

Do not put passwords in a Dockerfile `ENV`: they become part of the image, and anyone with the image can read them with `docker inspect`.

### Volumes

A **volume** is storage that Docker creates and manages. It lives **outside** any container, so it survives when containers are stopped, deleted or replaced with a newer image.

- **Docker chooses where it is stored.** `docker volume inspect` shows a `Mountpoint` like `/var/lib/docker/volumes/incrementor-data/_data`. On Docker Desktop for Mac, that path is inside Docker's Linux virtual machine, **not** on your Mac, so you cannot open it in Finder.
- **You refer to it by name**, like `incrementor-data`. Any container can use it with `--mount type=volume,src=incrementor-data,...`.
- **It starts empty**, unless the image already has files in the `target` folder (see [Volumes and non-root users](#volumes-and-non-root-users)).
- **Several containers can use it**, at the same time or one after another.

Volumes are the standard way to keep data such as:

- database files (PostgreSQL, MySQL, MongoDB)
- files that users upload
- caches you want to keep between runs

#### Look inside a volume

You cannot open a volume from your Mac, but any container can mount it. A tiny Alpine container is enough:

```sh
docker run --rm --mount type=volume,src=incrementor-data,target=/data alpine sh -c 'ls -la /data; cat /data/num.txt'
```

Measured here, after three runs: it lists `num.txt` (owned by `root`) and prints `2`.

### Managing volumes

| Command | What it does |
| --- | --- |
| `docker volume create incrementor-data` | Creates an empty volume. Not needed here: `docker run --mount` creates it if it is missing. |
| `docker volume ls` | Lists all volumes. See [Listing volumes](#listing-volumes-docker-volume-ls). |
| `docker volume inspect incrementor-data` | Shows details: driver, `Mountpoint`, creation date. |
| `docker volume rm incrementor-data` | Deletes the volume **and all its data**. The next run starts again from `0`. |
| `docker volume prune` | Deletes unused volumes. See the warning below. |

#### Listing volumes: docker volume ls

```sh
docker volume ls
```

Lists every volume on this machine. `docker volume list` does the same thing. Output on this machine (shortened):

```
DRIVER    VOLUME NAME
local     0a35fab2738ec3edc9ebda1fd80193ecb35a2beb8127ab9b5077eca8b3566ed5
local     2bc7811367bc8d134006f443a709e90e84506780821fd8087f87abb6d1f38ff6
...
local     incrementor-data
local     transcendence_avatar-data
local     transcendence_db-data
```

| Column | Meaning |
| --- | --- |
| `DRIVER` | Where the data is stored. `local` means on this machine, in Docker's storage area. |
| `VOLUME NAME` | The volume's name. A long random name (64 hexadecimal characters) means an **anonymous** volume. A readable name means a **named** volume. |

What the names tell you:

- **`incrementor-data`** was created by `--mount type=volume,src=incrementor-data` in [App 7](#run-the-incrementor-with-a-volume).
- **`transcendence_avatar-data`** and **`transcendence_db-data`** were created by Docker Compose. Compose names volumes `<project>_<volume>`, so these are the `avatar-data` and `db-data` volumes of a project called `transcendence`.
- **The random names** are anonymous volumes. Nothing in the list says which container or image made them.

On this machine, `docker volume ls` lists **87** volumes, and 80 of them are not used by any container. `docker system df` shows that volumes take **35.11 GB**, of which 33.85 GB (96%) is "reclaimable", meaning no container uses it. That does **not** mean the data is useless. It only means no container is attached to it right now.

`docker volume ls` does not show sizes. To see how big each volume is, and how many containers use it (`LINKS`), use:

```sh
docker system df -v
```

```
VOLUME NAME                 LINKS     SIZE
transcendence_db-data       1         66MB
transcendence_avatar-data   2         138.7kB
```

##### Options

| Option | What it does |
| --- | --- |
| `--quiet` (`-q`) | Prints only the names, one per line. Useful in scripts. |
| `--filter` (`-f`) | Shows only the volumes that match. You can use it several times; a volume must match all of them. |
| `--format` | Chooses which fields to print, with a Go template. |

##### Filters

| Filter | Shows | Example |
| --- | --- | --- |
| `name=<text>` | Volumes whose name **contains** the text. | `docker volume ls -f name=transcendence` |
| `dangling=true` | Volumes that **no container** uses, running or stopped. | `docker volume ls -f dangling=true` |
| `dangling=false` | Volumes that at least one container uses. | `docker volume ls -f dangling=false` |
| `label=<key>` or `label=<key>=<value>` | Volumes with that label. | `docker volume ls -f label=com.docker.compose.project=transcendence` |
| `driver=<name>` | Volumes that use that driver. | `docker volume ls -f driver=local` |

Tested on this machine:

```sh
docker volume ls --filter name=transcendence
```

```
DRIVER    VOLUME NAME
local     transcendence_avatar-data
local     transcendence_db-data
```

**"Dangling" does not mean "safe to delete".** A volume is dangling when no container is attached to it. `incrementor-data` is dangling after every run, because `--rm` deletes the container, but the volume still holds the counter you want to keep. `docker volume ls -f dangling=true` shows exactly what [`docker volume prune`](#managing-volumes) would delete, so check that list before pruning.

`transcendence_db-data` is **not** dangling, even though its container has stopped (`transcendence-db-1`, `Exited`). A stopped container still counts as using its volumes, until the container is removed.

##### Custom output: --format

`--format` takes a template with these fields: `{{.Name}}`, `{{.Driver}}`, `{{.Scope}}`, `{{.Mountpoint}}`, `{{.Labels}}`, and `{{.Label "key"}}` for one label. Start the template with `table` to get column headings:

```sh
docker volume ls --filter name=transcendence --format 'table {{.Name}}\t{{.Driver}}\t{{.Labels}}'
```

```
VOLUME NAME                 DRIVER    LABELS
transcendence_avatar-data   local     com.docker.compose.volume=avatar-data,com.docker.compose.project=transcendence,com.docker.compose.version=2.15.1
transcendence_db-data       local     com.docker.compose.volume=db-data,com.docker.compose.project=transcendence,com.docker.compose.version=2.15.1
```

The labels show that Docker Compose 2.15.1 created these volumes, for the `transcendence` project. Anonymous volumes have no labels (`null`).

##### Useful combinations

```sh
# Count all volumes
docker volume ls -q | wc -l

# Show only volumes that no container uses (what prune would delete)
docker volume ls -f dangling=true

# Show the containers, running or stopped, that use a volume
docker ps -a --filter volume=transcendence_db-data
```

Example output of `docker volume inspect`:

```json
[
    {
        "CreatedAt": "2026-11-03T17:54:18Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/incrementor-data/_data",
        "Name": "incrementor-data",
        "Options": null,
        "Scope": "local"
    }
]
```

- `Driver: local` means the data is stored on this machine. Other drivers can store volumes on network storage or in the cloud.
- `Scope: local` means only containers on this machine can use it.

**A volume in use cannot be deleted.** While a container uses it, `docker volume rm` fails with `volume is in use`. Stop and remove the container first.

**Anonymous volumes.** `docker volume ls` may show many volumes with long random names, like `0a35fab2738e...`. Those are **anonymous** volumes: volumes without a name, created by images that declare a `VOLUME` (for example database images) or by `-v /path` with no name. Named volumes like `incrementor-data` are easier to find and reuse. With `--rm`, Docker deletes a container's anonymous volumes, but never its named ones.

**Be careful with `docker volume prune`.** It deletes every volume that no container is using **right now**. On older Docker versions (Docker Engine 20.10, like on this machine), that includes **named** volumes, for example a project's database volume, even if you still need it. Newer versions (Docker 23 and later) remove only anonymous volumes unless you add `--all`. Check `docker volume ls` first, and delete volumes one by one with `docker volume rm` when in doubt.

### Volumes and non-root users

This image runs as `root`, so it can write to the new volume. Earlier sections recommend a non-root user, but that causes a problem here. Tested with `USER node` added to the Dockerfile:

```
file not found, writing "0" to /data/num.txt
Error: EACCES: permission denied, open '/data/num.txt'
```

The image has no `/data` folder, so Docker creates the volume's folder owned by `root`, and `node` cannot write to it.

**The fix:** create the folder in the image, owned by `node`, **before** `USER`:

```dockerfile
FROM node:20-alpine
RUN mkdir /data && chown node:node /data
COPY --chown=node:node . /src
WORKDIR /src
USER node
CMD ["node", "index.js"]
```

When a **new, empty** volume is mounted on a folder that exists in the image, Docker copies that folder's contents **and owner** into the volume. Tested here: the volume's `/data` is owned by UID `1000` (`node`), and the counter works again as `node`.

This copy happens only once, when the volume is empty. A volume that already has data keeps its own files and owners.

### Volumes, bind mounts and the container's own files

| | Container's own files | Volume | Bind mount |
| --- | --- | --- | --- |
| Example | `/src/data.txt` (no mount) | `--mount type=volume,src=incrementor-data,target=/data` | `--mount type=bind,source="$(pwd)"/dist,target=...` |
| Who manages it | The container | Docker | You (it is a folder on your machine) |
| Survives `docker rm` / `--rm` | No | **Yes** | Yes |
| Where it is stored | In the container's writable layer | Docker's storage area (`/var/lib/docker/volumes/`) | Any folder you choose |
| Easy to see and edit from your Mac | No | No (use a container to look inside) | **Yes** |
| Speed on Docker Desktop for Mac | Fast | Fast | Slower |
| Needs a path that exists on the machine | No | No | Yes |
| Good for | Temporary files | Data the app needs to keep: databases, uploads | Source code, built websites, config you edit |

**Rule of thumb:** if **you** need to work with the files, use a bind mount. If only the **app** needs them, and they must survive, use a volume.
