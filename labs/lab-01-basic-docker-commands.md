# Lab 01 — Basic Docker Commands

## What This Lab Covers

This lab covers the foundational Docker command line interface: the set of commands every cloud engineer uses to pull images, run containers, inspect what is running, and clean up resources. Before networking, storage, or orchestration make sense, these commands need to be second nature because they are the baseline for everything else in Docker.

In production cloud environments, these commands are not typically typed by hand. They live inside CI/CD pipelines, deployment scripts, and automation tools. Knowing exactly what each command does is what lets you debug a broken pipeline or investigate why a container is not behaving the way you expect.

This lab covers `docker run`, `docker ps`, `docker stop`, `docker rm`, `docker images`, `docker rmi`, `docker pull`, `docker exec`, `docker attach`, `docker logs`, `docker inspect`, `docker kill`, and `docker system prune`.

---

## How Docker Actually Works

A **container** is a running instance of a Docker image. It is an isolated process on your host machine with its own filesystem, networking, and process space. Containers are not virtual machines. They do not boot an operating system. They run a single process, and when that process exits, the container stops. That simplicity is what makes containers so useful. They are lightweight, fast to start, and only use the resources their one process actually needs.

An **image** is the blueprint used to create a container. It is a packaged snapshot of an application and everything it needs to run: code, runtime, and dependencies. The image itself never changes while a container is running from it.

**The container lifecycle** moves through a defined set of states: created, running, and exited. A container in an exited state still exists on disk and still takes up space. It is not gone just because it stopped running. You have to explicitly remove it with `docker rm`.

**Attached vs Detached Mode**
When you run a container without any flags, Docker runs it in attached mode by default. Your terminal gets tied to the container's output and you can see everything it is doing in real time. The problem is you cannot do anything else in that terminal while the container is running. The only way out is Ctrl+C, which stops the container entirely.
Attached mode is useful when you are testing something for the first time and want to see output right away.


Detached mode does the opposite. The -d flag starts the container in the background and immediately gives your terminal back. The container keeps running, you just are not watching it directly. Docker prints the container ID when it starts so you know it worked. From there you use docker ps to confirm it is running and docker logs to see its output whenever you need to.

Detached mode is the standard for anything you want to keep running. Attached mode is for quick testing and debugging.

 
**Container Names and IDs** Every container gets two identifiers assigned automatically: a long ID made up of letters and numbers, and a randomly generated name like `adoring_tesla` or `silly_samet`. Both work in any Docker command. When using the ID, you only need the first few characters, which is enough to tell it apart from other containers on your machine.

**Why Some Containers Exit Immediately** A container only stays running as long as the process inside it is running. If you start a container from an image that has no process defined, such as a base operating system image (i.e Ubuntu), it exits right away because there is nothing to keep it alive. 
To keep it running you have to give it something to do, like `docker run ubuntu sleep 100`.

**Docker Hub** This is Docker's default public registry where images are stored and shared. You can browse available images at [hub.docker.com](https://hub.docker.com)

---

## Commands Covered

**`docker run`**

`docker run` creates and starts a container from a specified image. If the image is not on your machine, Docker pulls it from Docker Hub first.

```
docker run nginx
```

The `-d` flag runs the container in the background so you get your terminal back immediately.

```
docker run -d nginx
```

The `--name` flag gives the container a name you choose instead of the random default.

```
docker run -d --name my-webserver nginx
```

The `--rm` flag tells Docker to automatically delete the container once it exits. Useful when you are testing and do not want stopped containers piling up.

```
docker run --rm ubuntu sleep 5
```

---

**`docker ps`**

`docker ps` lists all currently running containers along with their ID, image name, how long they have been up, and their name.

```
docker ps
```

The `-a` flag shows every container on your machine regardless of state: running, stopped, or exited. Without it, stopped containers do not show up at all. 
This matters because stopped containers still sit on disk and take up space even when they are not running.

```
docker ps -a
```

---

**`docker stop`**

`docker stop` tells a container to shut down cleanly. Docker gives it ten seconds to finish what it is doing. If it does not stop in time, Docker forces it to stop.

```
docker stop my-webserver
docker stop a1b2c
```

You can use either the container name or just the first few characters of the ID.

---

**`docker kill`**

`docker kill` stops a container immediately with no cleanup time. A common use case for this is when a container is completely unresponsive and `docker stop` is not working. For everything else, `docker stop` is the right choice.

```
docker kill my-webserver
```

---

**`docker rm`**

`docker rm` permanently removes a stopped or exited container from disk. It does not remove the image the container was built from, only the container itself.

```
docker rm my-webserver
```

You can remove several containers at once by listing their names or IDs separated by spaces.

```
docker rm a1b2 c3d4 e5f6
```

The `-f` flag removes a container even if it is still running, skipping the stop step.

```
docker rm -f my-webserver
```

Key: A container must be removed before you can delete the image it was built from.

---

**`docker images`**

`docker images` lists every image stored locally on your machine, along with its name, tag, ID, and size.

```
docker images
```

```
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    a1b2c3d4e5f6   2 weeks ago    64MB
alpine       latest    b2c3d4e5f6a7   3 weeks ago    7.33MB
ubuntu       latest    c3d4e5f6a7b8   4 weeks ago    77.8MB
```

---

**`docker rmi`**

`docker rmi` removes an image from your machine. The image cannot be in use by any container — running or stopped — before you can delete it. Remove the containers first, then the image.

```
docker rmi nginx
docker rmi alpine:latest
```

---

**`docker pull`**

`docker pull` downloads an image to your machine without running a container. This is useful when you want the image ready ahead of time so `docker run` does not have to wait for a download.

```
docker pull ubuntu
docker pull nginx:1.25.3
```

Adding a tag after the colon pulls a specific version instead of the latest.

---

**`docker exec`**

`docker exec` runs a command inside a container that is already running. It does not start a new container. Instead, it runs the command inside one that already exists. This is the main tool for looking inside a running container to check logs, inspect files, or test whether something is working.

```
docker exec my-webserver cat /etc/hosts
```

`-it` stands for interactive terminal. Paired together, they open an interactive terminal session inside the container so you can type commands directly into it. Without them, Docker would run the command and exit immediately with no output and no way to interact with it. You would be left staring at a blank terminal with nothing to work with. 
This is how you explore a running container's filesystem, check a config file, or test whether something is working from inside the container's environment.

```
docker exec -it my-webserver bash
```

---

**`docker attach`**

`docker attach` connects your terminal back to a container that is already running in the background. You will see its output in real time.

```
docker attach my-webserver
```

Pressing `Ctrl+C` while attached stops the container entirely, not just the session. To leave without stopping the container, press `Ctrl+P` then `Ctrl+Q`.

---

**`docker logs`**

`docker logs` shows the output a container has produced (regardless of whether it is currently running or has already stopped). When a container exits unexpectedly, this is the first place to look.

```
docker logs my-webserver
```

The `-f` flag streams the output live so you can watch it in real time.

```
docker logs -f my-webserver
```

The `--tail` flag limits the output to the last number of lines you specify.

```
docker logs --tail 50 my-webserver
```

---

**`docker inspect`**

`docker inspect` returns detailed information about a container or image in JSON format, such as network settings, environment variables, mounted storage, and more. Use it when you need to confirm exactly how a container was configured.

```
docker inspect my-webserver
```

---

**`docker system prune`**

`docker system prune` cleans up your machine in one command. It removes all stopped containers, unused images, unused networks, and unused build cache.

```
docker system prune
```

The `-a` flag extends the cleanup to all unused images, not just ones with no tag. The `--volumes` flag also removes unused storage volumes. The `-f` flag skips the confirmation prompt.

```
docker system prune -a --volumes -f
```

---

## AWS Connection

Every command in this lab is doing manually what AWS does automatically at scale. When Amazon ECS runs a container, it is pulling an image, starting it, monitoring its state, and cleaning up after it. This is the same sequence I just practiced by hand. It is still extremely important to understand the manual version first, so that the automated version is readable. When something breaks in ECS, knowing what `docker logs` and `docker inspect` expose is what tells you where to look. Lab 08 covers Amazon ECR, which is where images live in a production AWS environment instead of Docker Hub.

---

## Troubleshooting

**`docker run ubuntu` exited immediately and I had no idea why**
I ran the command expecting a terminal or some kind of output and got nothing. Running `docker ps` showed no containers. It took me a minute to understand that Ubuntu is just a base image and that there is no application inside it running by default. Once I understood that containers exit when their process exits, I ran `docker run ubuntu sleep 100` to keep it alive long enough to actually see it in `docker ps`.

**`docker rmi` failed with an error about the image being in use**
I tried to delete the nginx image after stopping the container and got an error. Running `docker ps -a` showed the stopped container was still sitting there. Docker will not delete an image if any container (even a stopped one) was built from it. I had to run `docker rm` on the container first, then `docker rmi` worked.

**I pressed `Ctrl+C` to detach from a container and it stopped the whole thing**
I had attached to a running container with `docker attach` and assumed `Ctrl+C` would just exit the session. It stopped the container entirely. The correct way to leave without stopping the container is `Ctrl+P` then `Ctrl+Q`. I did not know that existed until I had already killed a container I wanted to keep running.

**Removing multiple stopped containers one at a time felt slow**
After a few rounds of testing I had a handful of stopped containers sitting around. I was removing them one by one until I realized `docker rm` accepts multiple names or IDs in a single command separated by spaces. For a bigger cleanup, `docker system prune` removes everything at once.

---

## Key Observations

- I went into this assuming that stopping a container was the same as removing it. It is not. A stopped container is still on disk, still holds a reference to its image, and will block `docker rmi` from working. Running `docker ps -a` after every session is a habit I am building now.

- The first time I ran `docker run ubuntu`, I had no idea why it stopped. I expected `docker run ubuntu` to give me a running environment the way a virtual machine would. Understanding that a container exits when its process exits changed how I think about what Docker actually is.

- `docker logs` is the first place to look when something goes wrong. Before I knew it existed I was running `docker ps -a` and staring at an exited container with no idea what happened inside it.

- Detached mode felt strange at first because there is no confirmation that anything worked. Running `docker ps` right after `docker run -d` to verify the container is actually up is something I now do every time.
