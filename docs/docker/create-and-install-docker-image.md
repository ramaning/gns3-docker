# Docker Image Creation and Installation — Learner Guide

## Overview

This practical lab teaches you how to create a Docker image from a `Dockerfile`, build and test it, save it as a portable file, and load it on another Docker host.

You will also learn the Docker Registry workflow used to share images between systems.

## Learning Objectives

By the end of this lab, you should be able to:

- Explain the difference between a Docker image and a container.
- Create a simple application and Dockerfile.
- Build and tag a Docker image.
- Run and inspect a container.
- View container logs and troubleshoot basic problems.
- Stop, start, restart, and remove containers.
- Export an image with `docker save`.
- Import an image with `docker load`.
- Explain how Docker Registry push/pull works.

## 1. Prerequisites

You need:

- An Ubuntu/Linux system with Docker Engine installed.
- A user account with permission to run Docker commands.
- Basic Linux command-line knowledge.
- Internet access for the optional Registry section.

### Verify Docker

Run:

```bash
docker --version
docker info
```

If `docker info` reports that the Docker daemon is not running, start it with:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

If you receive a Docker socket permission error, either use `sudo` or configure the user for Docker according to your organization's security policy.

## 2. Docker Image vs. Container

A **Docker image** is a read-only package containing the application, runtime, libraries, configuration, and other files required to create a container.

A **container** is a running or stopped instance created from an image.

A useful way to remember the relationship is:

```text
Dockerfile  ->  Docker Image  ->  Docker Container
```

You can create many containers from the same image.

## 3. Create the Lab Project

Create a working directory:

```bash
mkdir -p ~/docker-lab/learner-app
cd ~/docker-lab/learner-app
```

Create a simple HTML page:

```bash
cat > index.html <<'EOF'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Docker Learner Lab</title>
</head>
<body>
    <h1>Docker Learner Lab</h1>
    <p>This application is running inside a Docker container.</p>
</body>
</html>
EOF
```

Check the file:

```bash
ls -l
cat index.html
```

## 4. Create the Dockerfile

Create a file named exactly `Dockerfile`:

```bash
cat > Dockerfile <<'EOF'
FROM python:3.12-alpine

WORKDIR /app

COPY index.html .

EXPOSE 8080

CMD ["python", "-m", "http.server", "8080"]
EOF
```

### Dockerfile Explanation

| Instruction | Purpose |
|---|---|
| `FROM` | Selects the base image used to build the image. |
| `WORKDIR` | Sets the working directory inside the image/container. |
| `COPY` | Copies files from the build context into the image. |
| `EXPOSE` | Documents the port that the application listens on. It does not publish the port by itself. |
| `CMD` | Defines the default command used when a container starts. |

The `python:3.12-alpine` image is used here because it provides Python on a small Alpine Linux base.

## 5. Build the Docker Image

From the directory containing the Dockerfile, run:

```bash
docker build -t learner-app:1.0 .
```

### Understanding the command

- `docker build` tells Docker to build an image.
- `-t learner-app:1.0` assigns the image a name and tag.
- `.` tells Docker to use the current directory as the build context.

The resulting image should be named:

```text
learner-app:1.0
```

## 6. Verify the Image

List local images:

```bash
docker images
```

You should see `learner-app` with tag `1.0`.

Inspect the image:

```bash
docker image inspect learner-app:1.0
```

You can also display the image history:

```bash
docker history learner-app:1.0
```

## 7. Run a Container

Start a container from the image:

```bash
docker run -d --name learner-app -p 8080:8080 learner-app:1.0
```

The command means:

- `-d` runs the container in detached mode.
- `--name learner-app` assigns the container a name.
- `-p 8080:8080` maps host TCP port 8080 to container TCP port 8080.
- `learner-app:1.0` is the image used to create the container.

Check the running container:

```bash
docker ps
```

## 8. Test the Application

From the Docker host, test the web service:

```bash
curl http://127.0.0.1:8080
```

You should receive the HTML content from `index.html`.

If you are accessing the application from another machine, use the Docker host's IP address:

```text
http://SERVER-IP:8080
```

## 9. Check Logs and Container Details

View the container logs:

```bash
docker logs learner-app
```

Follow the logs in real time:

```bash
docker logs -f learner-app
```

Inspect the container:

```bash
docker inspect learner-app
```

Display the container's IP address:

```bash
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' learner-app
```

## 10. Container Lifecycle

Stop the container:

```bash
docker stop learner-app
```

Start it again:

```bash
docker start learner-app
```

Restart it:

```bash
docker restart learner-app
```

List all containers, including stopped containers:

```bash
docker ps -a
```

Remove the container after stopping it:

```bash
docker rm learner-app
```

Removing a container does **not** remove the image.

Confirm that the image still exists:

```bash
docker images
```

## 11. Save the Docker Image to a File

Docker images can be packaged into a tar archive for transfer to another Docker host.

First make sure the image exists:

```bash
docker images learner-app
```

Save it:

```bash
docker save -o learner-app-1.0.tar learner-app:1.0
```

Verify the file:

```bash
ls -lh learner-app-1.0.tar
```

You can now transfer the tar file to another Docker host using your normal approved file-transfer method, such as `scp`.

Example:

```bash
scp learner-app-1.0.tar user@REMOTE-SERVER:/tmp/
```

Replace `user` and `REMOTE-SERVER` with the appropriate values for your lab.

## 12. Load the Image on Another Docker Host

On the destination Docker host, load the image:

```bash
docker load -i /tmp/learner-app-1.0.tar
```

Verify that it was imported:

```bash
docker images
```

You should see:

```text
learner-app   1.0
```

Run the imported image:

```bash
docker run -d --name learner-app -p 8080:8080 learner-app:1.0
```

Test it:

```bash
curl http://127.0.0.1:8080
```

This demonstrates that an image can be built on one Docker host, transferred, imported, and run on another host without rebuilding it.

## 13. Sharing Images with a Docker Registry

For larger environments, manually transferring tar files is not usually the preferred method. A Docker Registry allows images to be stored and retrieved over a network.

The general workflow is:

```text
Build -> Tag -> Push -> Registry -> Pull -> Run
```

### Tag an Image

A registry-qualified image name normally includes the registry hostname or namespace.

Example:

```bash
docker tag learner-app:1.0 REGISTRY/learner-app:1.0
```

### Push the Image

After authenticating to the registry:

```bash
docker push REGISTRY/learner-app:1.0
```

### Pull the Image on Another Host

```bash
docker pull REGISTRY/learner-app:1.0
```

Then run it:

```bash
docker run -d --name learner-app -p 8080:8080 REGISTRY/learner-app:1.0
```

Replace `REGISTRY` with the hostname, namespace, or registry path appropriate to your environment.

> **Security note:** Do not place registry passwords, access tokens, private keys, or other credentials inside a Dockerfile or commit them to Git.

## 14. Troubleshooting

### Docker command not found

Check whether Docker is installed:

```bash
docker --version
```

If it is not installed, install Docker Engine using the procedure approved for your Linux distribution.

### Docker daemon is not running

Check the service:

```bash
sudo systemctl status docker
```

Start it if required:

```bash
sudo systemctl start docker
```

### Permission denied on `/var/run/docker.sock`

Check the Docker service and your permissions. As a temporary diagnostic step, try:

```bash
sudo docker ps
```

If `sudo` works but the normal command does not, review your user's Docker group membership and your organization's security requirements.

### Port 8080 is already in use

Check what is using the port:

```bash
sudo ss -ltnp | grep ':8080'
```

You can use a different host port while keeping the container port at 8080:

```bash
docker run -d --name learner-app -p 8081:8080 learner-app:1.0
```

Then access:

```text
http://SERVER-IP:8081
```

### Container exits immediately

Check the container state and logs:

```bash
docker ps -a
docker logs learner-app
```

The logs usually provide the first indication of why the application stopped.

### Image not found

List local images:

```bash
docker images
```

Make sure the image name and tag are correct. For an image transferred from another host, confirm that `docker load` completed successfully.

### Build fails

Confirm that you are in the directory containing the Dockerfile:

```bash
pwd
ls -la
```

Then rebuild:

```bash
docker build -t learner-app:1.0 .
```

Read the first meaningful error in the build output rather than only the final error message.

## 15. Practical Learner Exercise

Complete the following exercise without copying the entire workflow from the previous sections.

### Task

Create and transfer a Docker image called `student-app` with tag `1.0`.

The application should return a simple web page containing your name and the statement:

```text
This application is running inside a Docker container.
```

### Requirements

1. Create a project directory.
2. Create the application file.
3. Create a Dockerfile.
4. Build `student-app:1.0`.
5. Verify the image exists.
6. Run a container from the image.
7. Test the application using `curl` or a web browser.
8. View the container logs.
9. Stop and remove the container.
10. Save the image as `student-app-1.0.tar`.
11. Transfer the tar file to another Docker host.
12. Load the image on the second host.
13. Verify the imported image.
14. Run the imported image.
15. Test the application again.

### Evidence of Completion

A successful submission should include:

- The Dockerfile.
- The application source file.
- Output showing the image exists.
- Output showing the container is running.
- Output from the application test.
- Evidence that the image was exported.
- Evidence that the image was loaded on the second Docker host.
- Evidence that the imported image runs successfully.

## 16. Knowledge Check

Answer these questions after completing the lab:

1. What is the difference between an image and a container?
2. What does the `-t` option do in `docker build`?
3. What does the final `.` mean in `docker build -t learner-app:1.0 .`?
4. Does `EXPOSE 8080` publish port 8080 to the host? Explain.
5. What is the purpose of `-p 8080:8080`?
6. What is the difference between `docker save` and `docker load`?
7. What happens to an image when a container created from it is removed?
8. When would you use a Docker Registry instead of transferring a tar file?

## 17. Quick Reference

### Images

```bash
docker images
docker image inspect IMAGE:TAG
docker history IMAGE:TAG
docker save -o image.tar IMAGE:TAG
docker load -i image.tar
```

### Containers

```bash
docker run -d --name NAME -p HOST_PORT:CONTAINER_PORT IMAGE:TAG
docker ps
docker ps -a
docker logs NAME
docker inspect NAME
docker stop NAME
docker start NAME
docker restart NAME
docker rm NAME
```

### Build

```bash
docker build -t IMAGE:TAG .
```

### Registry

```bash
docker tag IMAGE:TAG REGISTRY/IMAGE:TAG
docker push REGISTRY/IMAGE:TAG
docker pull REGISTRY/IMAGE:TAG
```

## Lab Completion Checklist

- [ ] Docker installation verified.
- [ ] Project directory created.
- [ ] Application created.
- [ ] Dockerfile created.
- [ ] Docker image built.
- [ ] Image verified.
- [ ] Container started.
- [ ] Application tested.
- [ ] Container logs checked.
- [ ] Container lifecycle commands tested.
- [ ] Image exported with `docker save`.
- [ ] Image transferred to another Docker host.
- [ ] Image imported with `docker load`.
- [ ] Imported image successfully run.
- [ ] Registry workflow understood.
- [ ] Practical exercise completed.
- [ ] Knowledge-check questions answered.
