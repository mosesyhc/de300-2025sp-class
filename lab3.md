# Lab 3: Docker and Github Review

### 🧱 What is a Docker Container?
A Docker container is a lightweight, portable, and isolated environment for running applications. You can think of it as a mini virtual computer that includes:
- The application

- All its dependencies (libraries, tools, etc.)

- An operating system user space, but not the full OS kernel

Containers share the host machine’s kernel, but remain isolated from each other — which makes them much faster and lighter than virtual machines.

### 📦 Container = Code + Environment
Imagine you're developing a Python app on your laptop and want to deploy it to a server or share it with a teammate. But:

- Your laptop has Python 3.10 and pandas 2.0

- The server has Python 3.7 and no pandas

- Your teammate uses Windows

A Docker container solves all that by packaging: 
- ✅ Your exact Python version
- ✅ Your libraries and dependencies
- ✅ Your file system and working directory

### Docker Container vs Virtual Machine (VM)

| Feature         | Docker Container                     | Virtual Machine                      |
|----------------|---------------------------------------|--------------------------------------|
| Boot Time      | Seconds                               | Minutes                              |
| Resource Usage | Light (shares host kernel)            | Heavy (includes full OS)             |
| Size           | MBs                                   | GBs                                  |
| Isolation      | Process-level                         | Full OS-level                        |
| Portability    | High                                  | Medium                               |
| Use Case       | Microservices, CI/CD, reproducible dev envs | Legacy systems, full OS sandboxing |



## 0. Installing Docker
### Windows/Mac
1. Download Docker Desktop from [Docker's official website](https://www.docker.com/products/docker-desktop/).
2. Follow the installation instructions provided.
3. Verify installation:
   ```bash
   docker --version
   ```

| Command                                | Description                             |
|----------------------------------------|-----------------------------------------|
| `docker ps`                            | List running containers                 |
| `docker ps -a`                         | List all containers (even stopped)      |
| `docker images`                        | List all images                         |
| `docker build -t <name> .`             | Build an image from Dockerfile          |
| `docker run <image>`                  | Run a container                         |
| `docker run -it <image> /bin/bash`     | Run with interactive terminal           |
| `docker exec -it <container_id> bash`  | Run a command in running container      |
| `docker stop <container_id>`           | Stop a container                        |
| `docker rm <container_id>`             | Remove a container                      |
| `docker rmi <image_id>`                | Remove an image                         |


## 1. 🐍 Creating a Docker Container to Run a Python Script

### 1.0 Create a dedicated directory (Optional)

In order to keep this process more clean, we recommend you make a new directory.

```bash
mkdir docker-tutorial
cd docker-tutorial
```

### 1.1 Create a Python Script
`script.py`

```{python}
import pandas as pd

df = pd.DataFrame({
    'Name': ['Alice', 'Bob'],
    'Age': [25, 30]
})

df.to_csv('output.csv', index=False)
print("CSV file created!")
```

### 1.2 Create a Dockerfile
`Dockerfile`
```{Dockerfile}
FROM python:3.10
WORKDIR /app
COPY script.py .
RUN pip install pandas
CMD ["python", "script.py"]
```

### 1.3 Build the Docker Image
```{bash}
docker build -t csv-generator .
```

### 1.4 Run the Container
```{bash}
docker run csv-generator
```
This will create the CSV file inside the container only, not on your host (i.e., your own computer).

### 1.5 🧠 Image vs Container
| Docker Image                  | Docker Container         |
|------------------------------|---------------------------|
| Blueprint                    | Live instance             |
| Read-only                    | Read/write                |
| Doesn’t run on its own       | Runs processes            |
| Can create many containers   | Based on one image        |


## 2. 📁 Sync Files Between Container and Host Machine
### 2.1 Create a Directory on Host
```{bash}
mkdir $(pwd)/csv-output
```

### 2.2 Update the Python Script

```{python}
import pandas as pd

df = pd.DataFrame({
    'Name': ['Alice', 'Bob'],
    'Age': [25, 30]
})

df.to_csv('output/output.csv', index=False)
print("CSV file created and saved to volume!")
```

### 2.3 Run With Volume Mount
```{bash}
docker run -v $(pwd)/csv-output:/app/output csv-generator
```

## 3. 🌟 Other Useful Concepts and Tips
### 3.1: Debug/Explore Inside a Container
```{bash}
docker run -it --entrypoint /bin/bash csv-generator
```
Then run:
```{bash}
python script.py
```

### 3.3: Cleaning Up Docker
```{bash}
docker system prune        # Remove all unused containers/images/volumes
docker container prune     # Remove stopped containers
docker image prune         # Remove unused images
```

## Intalling Docker on EC2
```{bash}
sudo apt update
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

## 🧑‍💻 Git + GitHub Tutorial (Beginner to Intermediate)
### 🧩 What is Git and GitHub?
- Git is a version control system that tracks code changes locally.
- GitHub is a cloud platform that hosts Git repositories and supports collaboration.

### 🧰 Part 1: Setting Up
#### ✅ 1.1 Install Git
On macOS:
```{bash}
brew install git
```

On Ubuntu (e.g., EC2):
```{bash}
sudo apt update
sudo apt install git
```


On Amazon Linux (e.g., EC2):
```{bash}
sudo yum update
sudo yum install git
```

### 🔑 Part 2: SSH Key Authentication (for GitHub)
#### ✅ 2.1 Generate SSH Key (on local or EC2)
```{bash}
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Accept default file location `(~/.ssh/id_rsa)`, preferrably don't set a passcode.
The next step is to copy your ssh key into your clipboard so that you can input it into github.

`pbcopy` is a command on macOS systems to copy a text into the clipboard. Linux has a similar command, however, we want to copy to the clipboard of our computer, not the ec2.

```{bash}
pbcopy < ~/.ssh/id_rsa.pub   # macOS
cat ~/.ssh/id_rsa.pub        # for manual copy (Linux)
```

#### ✅ 2.2 Add SSH Key to GitHub
- Go to GitHub → Settings → SSH and GPG keys

- Click New SSH key

- Paste the copied key

### 📥 Part 3: Clone a GitHub Repo
```{bash}
git clone git@github.com:your_username/FirstName_LastName_DE300.git
cd FirstName_LastName_DE300
```

### 🛠️ Part 4: Make Changes Locally
```{bash}
cd direction_to_your_downloaded_folder
mkdir lab3
```
Then copy paste the Dockerfile and script.py to the lab3 folder.

### 🔄 Part 5: Track, Commit, and Push Changes
```{bash}
git status                 # Check what changed
git add .                  # Stage all changes
git commit -m "created lab3 folder"   # Commit with message
git push origin main       # Push to GitHub (replace 'main' if needed)
```

### Part 6: Pull Changes from GitHub from EC2
ssh to your EC2, and enter the following commands on your EC2 instance
```{bash}
git pull origin main
```

### Part 7: Modified FILFI