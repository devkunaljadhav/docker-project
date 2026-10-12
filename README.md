# Docker Banking Demo

A small containerization and CI/CD demonstration project. It packages a static, banking-themed HTML page in Apache and includes a Docker Swarm stack and a Jenkins pipeline example.

> **Educational demo only.** The page is a mock UI, not a banking application. It has no authentication backend, and its form submits to a placeholder path. Do not enter real credentials or use this project to collect or process sensitive data.

## Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Repository layout](#repository-layout)
- [Run the web page locally](#run-the-web-page-locally)
- [Deploy the stack to Docker Swarm](#deploy-the-stack-to-docker-swarm)
- [Jenkins pipeline](#jenkins-pipeline)
- [Configuration reference](#configuration-reference)
- [Limitations](#limitations)
- [Author](#author)

## Overview

The project demonstrates a basic container workflow:

1. Build an Apache image from the included `Dockerfile`.
2. Serve `index.html` from the container.
3. Deploy a multi-service stack to Docker Swarm using `docker-compose.yml`.
4. Optionally use the example Jenkins pipeline to build and publish an image and deploy the stack.

The included web page is a single static mockup. The stack manifest describes four services—Internet Banking, Mobile Banking, Insurance, and Loan—but the repository does not contain separate application implementations for them.

## Architecture

The diagram illustrates the intended GitHub → Jenkins → Docker Hub → Docker Swarm workflow.

![Docker Swarm CI/CD architecture](<Docker Swarm CI_CD Architecture Diagram.png>)

## Repository layout

| Path | Purpose |
| --- | --- |
| `Dockerfile` | Builds an Ubuntu-based image with Apache and copies in `index.html`. |
| `index.html` | Static banking-themed mock login page. |
| `docker-compose.yml` | Docker Swarm stack definition for four services. |
| `JenkinsPipeline` | Declarative Jenkins pipeline example. Configure Jenkins to use this file as the pipeline script. |
| `PROCESS` | Original environment setup and deployment notes. |
| `Docker Swarm CI_CD Architecture Diagram.png` | CI/CD and Swarm architecture diagram. |

## Run the web page locally

Requirements: [Docker Engine](https://docs.docker.com/engine/) installed and running.

From the repository root, build and start the single-page demo:

```sh
docker build -t docker-banking-demo .
docker run --rm -d --name docker-banking-demo -p 8080:80 docker-banking-demo
```

Open [http://localhost:8080](http://localhost:8080) in a browser. Stop the container with:

```sh
docker stop docker-banking-demo
```

## Deploy the stack to Docker Swarm

Requirements:

- Docker Engine on the manager and worker hosts.
- A Swarm cluster with a manager node.
- The images referenced in `docker-compose.yml` available to the Swarm nodes through Docker Hub (or another configured registry).
- Network access to the service ports `81`–`84`.

Initialize a Swarm on the intended manager if the cluster has not already been created:

```sh
docker swarm init
```

Deploy the stack from the repository root:

```sh
docker stack deploy --compose-file docker-compose.yml bank
```

Check the stack and service rollout:

```sh
docker stack ls
docker stack services bank
docker stack ps bank
```

When the services are running, access them at the Swarm hosts on the published ports:

| Service | Published port |
| --- | ---: |
| Internet Banking | `81` |
| Mobile Banking | `82` |
| Insurance | `83` |
| Loan | `84` |

Remove the stack when it is no longer needed:

```sh
docker stack rm bank
```

`docker-compose.yml` requests three replicas for each service. The `deploy` settings are intended for Swarm and are not applied by a regular `docker compose up` deployment.

## Jenkins pipeline

`JenkinsPipeline` demonstrates checkout, image build, tagging, push, and Swarm deployment. To use it, create a Jenkins Pipeline job configured to load this file from source control or paste its contents into the pipeline definition.

The example requires a Jenkins agent with Git and Docker installed, access to the Swarm manager, and externally configured values for `image`, `repo`, and `password`. The pipeline currently hard-codes a checkout URL that differs from this repository's URL; update it to the correct repository before running.

**Credential warning:** the example logs in using `docker login -p`. Do not use that pattern with a real registry credential. Configure a Jenkins credential and use the Jenkins Docker/credentials mechanisms to authenticate instead. Keep registry credentials out of source code and build logs.

The pipeline deploys the same Swarm manifest, so the referenced service images must already exist in the registry and be pullable by the Swarm nodes. The included Dockerfile builds only the static Apache page; it does not build four distinct banking services.

## Configuration reference

The current `docker-compose.yml` uses these image names and published ports:

| Service | Image | Host port → container port |
| --- | --- | --- |
| `internetbanking` | `kunaljadhav1625/ib-image:latest` | `81:80` |
| `mobilebanking` | `kunaljadhav1625/mb-image:latest` | `82:80` |
| `insurance` | `kunaljadhav1625/insurance-image:latest` | `83:80` |
| `loan` | `kunaljadhav1625/loan-image:latest` | `84:80` |

Change the image references in the manifest if publishing to a different registry or namespace. Each service is configured with three replicas and a mount target under `/var/lib`; these are container paths, not explicitly named host-persistent volumes.

## Limitations

- The login form is a front-end mockup only. There is no server-side authentication, and its form target is a placeholder.
- The four Swarm services are registry image references, not four application implementations in this repository.
- `JenkinsPipeline` is an example, not a ready-to-run pipeline: its checkout URL and required variables/credentials must be configured for your Jenkins environment.
- This project does not include production security, data persistence, TLS, monitoring, or application-level tests.

## Author

**Kunal Jadhav**

- **GitHub:** [@devkunaljadhav](https://github.com/devkunaljadhav)
- **LinkedIn:** [devkunaljadhav](https://www.linkedin.com/in/devkunaljadhav)
- **Portfolio:** [kunalcloud.shop](https://kunalcloud.shop)
