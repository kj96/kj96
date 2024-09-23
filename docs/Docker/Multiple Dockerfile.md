

## Example docker project: faceswap

![[Pasted image 20240921135422.png]]

When you have two Dockerfiles, one for GPU support (`Dockerfile.gpu`) and one for CPU support (`Dockerfile.cpu`), there are a few good approaches you can follow. You can either distinguish between the two Dockerfiles by specifying them in the `docker-compose.yml` file or by creating separate directories for each Dockerfile.

Below are the best approaches, along with their pros and cons:

---

## **Approach 1: Specify Dockerfile in `docker-compose.yml`**

You can directly specify the `Dockerfile` in the `docker-compose.yml` file for each service. This allows you to avoid creating multiple directories, keeping your structure simple and organized.

### Example `docker-compose.yml` with Multiple Dockerfiles:

```yaml
version: '3'
services:
  faceswap_cpu:
    build:
      context: .
      dockerfile: Dockerfile.cpu
    volumes:
      - /path/to/local/data:/faceswap/data
    stdin_open: true
    tty: true

  faceswap_gpu:
    build:
      context: .
      dockerfile: Dockerfile.gpu
    volumes:
      - /path/to/local/data:/faceswap/data
    stdin_open: true
    tty: true
```

### Key Points:
- **`context: .`**: Refers to the current directory, and the `dockerfile` attribute specifies which Dockerfile to use.
- **No need for multiple directories**: You can keep both Dockerfiles in the root of the project and simply specify which one to use for each service in the `docker-compose.yml` file.

#### **Pros**:
- **Simple structure**: No need for multiple directories, and the `docker-compose.yml` handles the Dockerfile specification.
- **Single `docker-compose.yml`**: Both the CPU and GPU builds are in the same compose file, making it easier to manage.

#### **Cons**:
- **If you have other shared files (like requirements)**, it may lead to confusion if both Dockerfiles reference different dependencies. This can be solved by better organization of shared files.

---

## **Approach 2: Use Separate Directories for CPU and GPU**

This approach involves placing each Dockerfile in a separate directory (`cpu/` and `gpu/`) and referencing those directories in your `docker-compose.yml`.

### Directory Structure:
```
project-root/
│
├── cpu/
│   └── Dockerfile    # Rename to Dockerfile (no need for .cpu)
│
├── gpu/
│   └── Dockerfile    # Rename to Dockerfile (no need for .gpu)
│
└── docker-compose.yml
```

### Example `docker-compose.yml`:

```yaml
version: '3'
services:
  faceswap_cpu:
    build:
      context: ./cpu
    volumes:
      - /path/to/local/data:/faceswap/data
    stdin_open: true
    tty: true

  faceswap_gpu:
    build:
      context: ./gpu
    volumes:
      - /path/to/local/data:/faceswap/data
    stdin_open: true
    tty: true
```

### Key Points:
- **`context: ./cpu` and `context: ./gpu`**: These specify the directories containing the respective Dockerfiles.
- In this approach, each service (`faceswap_cpu` and `faceswap_gpu`) will have its own context directory.

#### **Pros**:
- **Better organization**: Each version of your build (CPU and GPU) is organized in separate directories.
- **Less ambiguity**: No need to specify the `dockerfile` option inside `docker-compose`, as each directory contains only one Dockerfile.
- **Can have different files per environment**: If CPU and GPU builds need different dependencies or files, this approach makes it easier to manage.

#### **Cons**:
- **Slightly more complex structure**: You need to manage multiple directories, which might feel more cluttered if the project is small.

---

## **Approach 3: Multiple Compose Files**

Another possible approach is to use multiple `docker-compose` files, one for CPU and one for GPU. This can be useful if the configurations for CPU and GPU are more different.

### Example File Structure:
```
project-root/
│
├── Dockerfile.cpu
├── Dockerfile.gpu
├── docker-compose.cpu.yml
└── docker-compose.gpu.yml
```

### Example `docker-compose.cpu.yml`:

```yaml
version: '3'
services:
  faceswap:
    build:
      context: .
      dockerfile: Dockerfile.cpu
    volumes:
      - /path/to/local/data:/faceswap/data
    stdin_open: true
    tty: true
```

### Example `docker-compose.gpu.yml`:

```yaml
version: '3'
services:
  faceswap:
    build:
      context: .
      dockerfile: Dockerfile.gpu
    volumes:
      - /path/to/local/data:/faceswap/data
    stdin_open: true
    tty: true
```

### Running the Compose Files:

- For CPU:  
  ```bash
  docker-compose -f docker-compose.cpu.yml up
  ```

- For GPU:  
  ```bash
  docker-compose -f docker-compose.gpu.yml up
  ```

#### **Pros**:
- **Clear separation**: Different `docker-compose.yml` files for different environments.
- **Modularity**: Easy to maintain if CPU and GPU configurations change significantly over time.

#### **Cons**:
- **Multiple compose files**: It adds a little more complexity in managing multiple compose files.

---

## **Which Approach Should You Use?**

### **1. Single Compose File with Dockerfile Specification**:
- **Use this approach if**: You want to keep a simple directory structure and manage both CPU and GPU configurations in one file.
  
### **2. Separate Directories for CPU and GPU**:
- **Use this approach if**: You want a clean separation of Dockerfile contexts and dependencies, especially if CPU and GPU builds have significantly different needs.

### **3. Multiple Compose Files**:
- **Use this approach if**: You want full modularity and flexibility, especially if CPU and GPU require very different runtime configurations or services. It also helps if you want to quickly switch between environments.

---

### **Recommended Approach**:
For most typical use cases, **Approach 1** (Single `docker-compose.yml` with Dockerfile specification) is usually the best balance between simplicity and flexibility. It keeps your structure clean and allows you to easily manage both CPU and GPU builds.