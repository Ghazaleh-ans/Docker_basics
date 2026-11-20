# Docker_basics

This repository has my notes and example projects from the course **Complete Intro to Containers, v2** by Brian Holt. Each folder is one small project. Each project shows one container idea, from a first `Dockerfile` to an app with three services on Kubernetes.

The full step-by-step guide is in [docs/README.md](docs/README.md).

## What this repository teaches

- **Container basics**: what images and containers are, and how to run, inspect, stop and remove them. Also how to scan an image for known security problems with `docker scout`.
- **Dockerfiles**: `FROM`, `WORKDIR`, `COPY`, `RUN` and `CMD`. Also how to run the app as a non-root user, install packages with `npm ci`, and use `.dockerignore`.
- **Faster builds**: how the layer cache works, and why you copy `package.json` before the rest of the code.
- **Smaller and safer images**: Alpine Linux, multi-stage builds and distroless images, and the trade-offs of each one.
- **Static websites**: build a site with Node.js, then serve the files with nginx.
- **Data**: bind mounts, volumes and environment variables.
- **Dev containers**: work on an app inside a container from VS Code, with Dependabot for updates.
- **Networks**: bridge networks, and how containers find each other by name through Docker's DNS.
- **Docker Compose**: start a web app, an API and a database with one file, and scale a service with `--scale`.
- **Kubernetes**: pods, Deployments, Services and `kubectl` on a local cluster (Docker Desktop or minikube). Also how Kompose converts a Compose file to Kubernetes files.

## Projects

| Folder | What it shows | Guide |
|---|---|---|
| `build-a-nodejs-app/` | A first `Dockerfile` for a Node.js web server | [App 1](docs/README.md#app-1-build-a-nodejs-app) |
| `build-a-more-complicated-nodejs-app/` | npm dependencies, the layer cache and `.dockerignore` | [App 2](docs/README.md#app-2-build-a-more-complicated-nodejs-app) |
| `alpine-linux/` | A smaller base image with Alpine Linux | [App 3](docs/README.md#app-3-alpine-linux) |
| `multi-stage-builds/` | Build in one stage, run in a smaller stage | [App 4](docs/README.md#app-4-multi-stage-builds) |
| `distroless/` | An image with only the runtime and no shell | [App 5](docs/README.md#app-5-distroless) |
| `static-asset-project/` | An Astro website served by nginx | [App 6](docs/README.md#app-6-static-asset-project) |
| `docker-volume/` | A counter that keeps its data in a volume | [App 7](docs/README.md#app-7-docker-volume) |
| `dev-containers/` | A Rails app in a VS Code dev container | [Dev Containers](docs/README.md#dev-containers-dev-containers) |
| `mongo-app/` | An app and MongoDB on a bridge network | [App 8](docs/README.md#app-8-mongo-app) |
| `docker-compose/` | Web, API and MongoDB with Docker Compose | [App 9](docs/README.md#app-9-docker-compose) |
| `kubernetes/` | The App 9 project on Kubernetes, made with Kompose | [Kubernetes](docs/README.md#kubernetes) |

## What you need

- [Docker Desktop](https://www.docker.com/products/docker-desktop/), or Docker Engine on Linux.
- For the Kubernetes project: `kubectl`, a local cluster (Docker Desktop or minikube) and `kompose`.

## Resources

- [Course website](https://containers-v2.holt.courses/): the written lessons of Complete Intro to Containers, v2.
- [Video course](https://master.dev/courses/complete-intro-containers-v2/): the video lessons of the same course.
- [Course project files](https://github.com/btholt/project-files-for-complete-intro-to-containers-v2): Brian Holt's original code for the projects.
