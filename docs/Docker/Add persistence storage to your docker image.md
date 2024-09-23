
## Example docker project: faceswap

```
(animation) ➜  faceswap git:(master) cat Dockerfile.cpu
FROM ubuntu:22.04

# To disable tzdata and others from asking for input
ENV DEBIAN_FRONTEND noninteractive
ENV FACESWAP_BACKEND cpu

RUN apt-get update -qq -y
RUN apt-get upgrade -y
RUN apt-get install -y libgl1 libglib2.0-0 python3 python3-pip python3-tk git

RUN ln -s $(which python3) /usr/local/bin/python

RUN git clone --depth 1 --no-single-branch https://github.com/deepfakes/faceswap.git
WORKDIR "/faceswap"

RUN python -m pip install --upgrade pip
RUN python -m pip --no-cache-dir install -r ./requirements/requirements_cpu.txt

CMD ["/bin/bash"]

```



To modify the Dockerfile to work with a persistent volume from your local storage, you typically do not specify the volume directly in the `Dockerfile`. Docker's philosophy separates the concerns of building the image and setting up the environment (volumes, networks, etc.) when running the container.

However, if you'd like to automate the mounting of a persistent volume when you run the container, you'll handle it in the `docker-compose` file or during the `docker run` command, rather than in the Dockerfile itself.

Here’s how you can incorporate this functionality:

### 1. **Persistent Volume with Docker Run Command**

No modification in  `Dockerfile` (No Changes Needed for Volume)

Your Dockerfile remains unchanged, as volumes are not specified inside the Dockerfile. The Dockerfile should focus only on installing and configuring the necessary software.

```dockerfile
FROM ubuntu:22.04

# To disable tzdata and others from asking for input
ENV DEBIAN_FRONTEND noninteractive
ENV FACESWAP_BACKEND cpu

RUN apt-get update -qq -y
RUN apt-get upgrade -y
RUN apt-get install -y libgl1 libglib2.0-0 python3 python3-pip python3-tk git

RUN ln -s $(which python3) /usr/local/bin/python

RUN git clone --depth 1 --no-single-branch https://github.com/deepfakes/faceswap.git
WORKDIR "/faceswap"

RUN python -m pip install --upgrade pip
RUN python -m pip --no-cache-dir install -r ./requirements/requirements_cpu.txt

CMD ["/bin/bash"]
```

### >  **Docker Run Command**
Instead of modifying the `Dockerfile`, you can mount a volume during the `docker run` command. Here’s how to do it:

```bash
docker run -it --rm -v /path/to/local/data:/faceswap/data faceswap:latest
```

In this example:
- `/path/to/local/data` is the directory on your host machine.
- `/faceswap/data` is the directory inside the container where your data will be stored and accessed persistently.

### 2. **Use `docker-compose` for Persistence**
If you want to simplify the process or ensure volume persistence across multiple containers, you can use a `docker-compose.yml` file. Here's how you can set up `docker-compose` to automatically mount the persistent volume.

#### Create a `docker-compose.yml` File

```yaml
version: '3'
services:
  faceswap_cpu:
    build:
      context: .
      dockerfile: Dockerfile.cpu  # Refers to your CPU-specific Dockerfile
    volumes:
      - ./:/srv                  # Mount current directory to `/srv` in container
      - /tmp/.X11-unix:/tmp/.X11-unix  # Mount X11 socket for GUI
      - /path/to/local/data:/faceswap/data   # Persistent volume for storing data
    environment:
      - DISPLAY=${DISPLAY}        # Pass the DISPLAY variable for graphical display
    stdin_open: true              # Allows for interactive terminal
    tty: true                     # Allocates a pseudo-TTY

# Define the persistent volume
volumes:
  faceswap_data:
    driver: local

```

### 4. **Build and Run with Docker Compose**
To build and run this setup using `docker-compose`, do the following steps:

1. **Build the image**:

```bash
docker-compose -f docker-compose.cpu.yml build --build-arg IMAGE_NAME=faceswap-cpupv
```

2. **Run the container with the mounted volume**:

```bash
xhost +local: && docker-compose -f docker-compose.cpu.yml up
```

### 5. **Verify the Volume is Working**
Inside the running container, you can verify that the volume is mounted and persistent by checking the `/faceswap/data` directory.

```bash
docker exec -it <container_name> /bin/bash
ls /faceswap/data
```

This command will let you check the contents of the mounted volume from the container.

---

### Summary:
- The `Dockerfile` doesn't handle volumes. Instead, volumes are set during container runtime using the `docker run` command or a `docker-compose` configuration.
- You can mount your local storage as a persistent volume using either the `-v` option in the `docker run` command or in a `docker-compose.yml` file.
