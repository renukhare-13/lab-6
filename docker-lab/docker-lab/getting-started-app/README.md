# Lab 6 – Docker & Containerization

**Author:** Renu Khare

In this lab I installed Docker Desktop on Windows, wrote a Dockerfile to containerize a Node.js to-do app, ran a MySQL server inside a Docker container, and connected to it from MySQL Workbench.

## Topics Covered

- Containerization vs. virtualization
- Virtual machines and container images
- Docker Desktop and Docker images
- Port mapping from a container to the host machine

## Repository Structure

```
lab-6-docker/
├── README.md
├── Dockerfile
├── package.json
├── spec/                       # Tests
├── src/                        # App source (Express + SQLite/MySQL)
└── screenshots/
    ├── lab_6.png        # To-do app
    ├── LAB 6.png        # MySQL command line + docker ps
    ├── LAB -6.png       # Docker Desktop
    └── lab6.png         # MySQL Workbench
```

## Prerequisites

- Windows 10/11 with WSL 2
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [MySQL Workbench](https://dev.mysql.com/downloads/workbench/)
- Git

## Task 1 – Install Docker Desktop

Installed Docker Desktop and confirmed it works from PowerShell:

```powershell
docker --version
```

## Task 2 – Dockerfile: Build and Run the App

The app is the to-do app from the official Docker [getting started guide](https://docs.docker.com/get-started/02_our_app/).

### Dockerfile

```dockerfile
# syntax=docker/dockerfile:1
FROM node:lts-alpine
WORKDIR /app
COPY . .
RUN npm install --omit=dev
EXPOSE 3000
CMD ["node", "src/index.js"]
```

| Instruction | Purpose |
|---|---|
| `FROM node:lts-alpine` | Lightweight Node.js LTS base image |
| `WORKDIR /app` | Working directory inside the container |
| `COPY . .` | Copies the app source into the image |
| `RUN npm install --omit=dev` | Installs production dependencies (npm is used because the current Node image no longer includes Yarn) |
| `EXPOSE 3000` | Documents the port the app listens on |
| `CMD [...]` | Starts the app when the container runs |

### Build and Run

```powershell
docker build -t getting-started .
docker run -dp 127.0.0.1:3000:3000 --name todo-app getting-started
```

Then open **http://localhost:3000**.

![To-do app running](<screenshots/lab_6.png>)

## Task 3 – MySQL Server in Docker

```powershell
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -p 3307:3306 -d mysql:8.0
```

`-p 3307:3306` maps port **3307** on my laptop to port 3306 inside the container. I used host port 3307 instead of 3306, and the `mysql:8.0` image instead of `mysql:latest`, to avoid connection issues on my machine.

Verified the server from inside the container:

```powershell
docker exec -it some-mysql mysql -uroot -pmy-secret-pw
```

```sql
SHOW DATABASES;
```

![MySQL command line and running containers](<screenshots/LAB 6.png>)

Both containers running in Docker Desktop:

![Docker Desktop running containers](<screenshots/LAB -6.png>)

## Task 4 – Connect with MySQL Workbench

| Setting | Value |
|---|---|
| Hostname | `127.0.0.1` |
| Port | `3307` |
| Username | `root` |
| Password | `my-secret-pw` |

![MySQL Workbench connected to the Docker MySQL server](<screenshots/lab6.png>)

## Task 5 – Sync to GitHub

Pushed the Dockerfile, app source, and screenshots to this repository.

## Troubleshooting

- **Workbench error 10061 (can't connect):** the container wasn't running or the port didn't match. Check `docker ps` and use the host port shown in the PORTS column.
- **`yarn: not found` during build:** replaced `yarn install --production` with `npm install --omit=dev`.
- **Container name already in use:** remove the old one with `docker rm -f <name>` and run again.

## Useful Commands

```powershell
docker ps                 # list running containers
docker ps -a              # list all containers
docker images             # list images
docker stop some-mysql    # stop the MySQL container
docker start some-mysql   # start it again
docker logs some-mysql    # view container logs
```

## References

- [Docker Desktop for Windows – install guide](https://docs.docker.com/desktop/setup/install/windows-install/)
- [Docker getting started – containerize an application](https://docs.docker.com/get-started/02_our_app/)
- [MySQL image on Docker Hub](https://hub.docker.com/_/mysql)
