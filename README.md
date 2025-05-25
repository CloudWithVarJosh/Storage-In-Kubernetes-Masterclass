# Storage in Kubernetes MASTERCLASS

## Video reference for the MASTERCLASS is the following:

[![Watch the video](https://img.youtube.com/vi/5t3yHFoqK6g/maxresdefault.jpg)](https://www.youtube.com/watch?v=5t3yHFoqK6g&t=12s&ab_channel=CloudWithVarJosh)

---
## ⭐ Support the Project  
If this **repository** helps you, give it a ⭐ to show your support and help others discover it! 

---
## Table of Contents

* [Introduction](#introduction)
* [Understanding Storage in Docker](#understanding-storage-in-docker)
  * [Layers in a Dockerfile](#layers-in-a-dockerfile)
  * [Example: Layers in Our Dockerfile](#example-layers-in-our-dockerfile)
  * [How to Inspect Image Layers](#how-to-inspect-image-layers)
  * [How Docker Containers Use Image Layers](#how-docker-containers-use-image-layers)
  * [Storage Calculation Example](#storage-calculation-example)
  * [Handling Data Loss in Writable Layers](#handling-data-loss-in-writable-layers)
  * [Need for Persistent Storage](#need-for-persistent-storage)
* [Storage in Docker: How Does It Work?](#storage-in-docker-how-does-it-work)
  * [1. Storage Drivers](#1-storage-drivers)
  * [2. Volume Drivers](#2-volume-drivers)
* [How Docker Storage Works on macOS, Windows, and Linux](#how-docker-storage-works-on-macos-windows-and-linux)
* [Understanding Docker Volumes](#understanding-docker-volumes)
  * [Where Are Docker Volumes Stored?](#where-are-docker-volumes-stored)
* [Demo: With Persistent Storage (Docker Volume)](#demo-with-persistent-storage-docker-volume)
  * [Note for macOS and Windows Users](#note-for-macos-and-windows-users)
* [Types of Docker Volumes: Which to Use When?](#types-of-docker-volumes-which-to-use-when)
* [Conclusion](#conclusion)
* [Kubernetes Core and Extended Architecture](#kubernetes-core-and-extended-architecture)
  * [Kubernetes Core Architecture](#kubernetes-core-architecture)
  * [Kubernetes Extensions: Beyond the Core](#kubernetes-extensions-beyond-the-core)
    * [1. Plugins](#1-plugins)
    * [2. Add-Ons](#2-add-ons)
    * [3. Third-Party Extensions](#3-third-party-extensions)
  * [Deep Dive into Kubernetes Interfaces](#deep-dive-into-kubernetes-interfaces)
  * [Why Kubernetes Uses a Plugin-Based Architecture](#why-kubernetes-uses-a-plugin-based-architecture)
  * [Key Takeaways](#key-takeaways)
* [Ephemeral Storage](#ephemeral-storage)
  * [emptyDir](#emptydir)
  * [Demo: emptyDir](#demo-emptydir)
* [Downward API](#downward-api)
  * [Demo: downwardAPI - Environment variables](#demo-downwardapi---environment-variables)
  * [Demo: downwardAPI - Files via Volumes](#demo-downwardapi---files-via-volumes)
* [ConfigMaps & Secrets (Preview)](#configmaps--secrets-preview)
* [Evolution of Storage in Kubernetes: From In-Tree to CSI Drivers](#evolution-of-storage-in-kubernetes-from-in-tree-to-csi-drivers)
  * [1. In-Tree Volume Plugins: The Legacy Model](#1-in-tree-volume-plugins-the-legacy-model)
  * [2. Container Storage Interface (CSI): The Modern Standard](#2-container-storage-interface-csi-the-modern-standard)
  * [Special Cases: hostPath and local Storage](#special-cases-hostpath-and-local-storage)
  * [Transitioning from In-Tree Drivers](#transitioning-from-in-tree-drivers)
* [Persistent Storage](#persistent-storage)
  * [hostPath](#hostpath)
  * [Understanding hostPath in KIND: How Storage Works Under the Hood](#understanding-hostpath-in-kind-how-storage-works-under-the-hood)
  * [Demo: hostPath](#demo-hostpath)
  * [Verification Across Pods on the Same Node](#verification-across-pods-on-the-same-node)
* [Persistent Volumes (PVs) & Persistent Volume Claims (PVCs)](#persistent-volumes-pvs--persistent-volume-claims-pvcs)
  * [Understanding Scope & Relationships of PV and PVC in Kubernetes](#understanding-scope--relationships-of-pv-and-pvc-in-kubernetes)
  * [1-to-1 Binding Relationship Between PVC and PV](#1-to-1-binding-relationship-between-pvc-and-pv)
  * [Kubernetes Persistent Storage Flow (Manual Provisioning)](#kubernetes-persistent-storage-flow-manual-provisioning)
* [Access Modes in Kubernetes Persistent Volumes](#access-modes-in-kubernetes-persistent-volumes)
  * [Explanation of Storage Types](#explanation-of-storage-types)
  * [Key Differences: Block Storage vs. File Storage](#key-differences-block-storage-vs-file-storage)
* [Reclaim Policies in Kubernetes](#reclaim-policies-in-kubernetes)
* [PVC and PV Binding Conditions](#pvc-and-pv-binding-conditions)
* [Working with ConfigMaps](#working-with-configmaps)
  * [What and Why of ConfigMaps](#what-and-why-of-configmaps)
  * [Demo: Injecting Environment Variables into Pods](#demo-injecting-environment-variables-into-pods)
  * [Demo 1: ConfigMaps as Environment Variables](#demo-1-configmaps-as-environment-variables)
  * [Demo 2: ConfigMaps as Configuration Files (Volumes)](#demo-2-configmaps-as-configuration-files-volumes)
* [Working with Secrets](#working-with-secrets)
  * [What and Why of Kubernetes Secrets](#what-and-why-of-kubernetes-secrets)
  * [Demo 1: Secrets as Environment Variables](#demo-1-secrets-as-environment-variables)
  * [Demo 2: Secrets as Volumes](#demo-2-secrets-as-volumes)
* [Conclusion](#conclusion-1)
* [References](#references)

---

### Introduction

This MASTERCLASS on **Storage in Kubernetes** provides a deep and structured walkthrough of storage concepts starting from Docker to advanced Kubernetes volumes. Whether you're preparing for the **CKA exam** or aiming to gain a robust understanding of containerized storage in production environments, this session offers a unified view of how ephemeral and persistent storage is managed.

The session begins by exploring how Docker handles image layers, writable layers, and the use of storage and volume drivers. It then transitions into Kubernetes, covering core and modular architecture, ephemeral storage mechanisms like `emptyDir`, the use of `hostPath`, and the lifecycle of Persistent Volumes (PVs) and Persistent Volume Claims (PVCs). Special attention is given to ConfigMaps, Secrets, reclaim policies, access modes, volume modes, and the CSI plugin architecture that powers modern Kubernetes storage management.

This content merges Days 24 to 28 of the **CKA 2025 course**, offering hands-on demos, YAML manifests, and clear explanations to ensure learners can apply concepts directly to real-world Kubernetes clusters.

---

### **Understanding Storage in Docker**

![Alt text](/images/24a.png)

Imagine we have a host system—this could be a VM or your laptop—that has the **Docker Engine** installed. We’re hosting three containers on this host: **Container 1**, **Container 2**, and **Container 3**. All these containers are based on an image created using the following `Dockerfile`:

```dockerfile
FROM python:3.9-slim
WORKDIR /app
ADD app.py app.py
RUN pip install flask
EXPOSE 5000
CMD ["python", "app.py"]
```

We’ve already discussed this `Dockerfile` in **Day-3’s lecture**. You can revisit the lecture for a recap:
- **YouTube link**: [Day-3 Lecture](https://youtu.be/MQ8fYqZwiQs)
- **GitHub link**: [Day-3 Resources](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2003)

The image created from this `Dockerfile` is named **`my-image`** and has a size of approximately **50MB** (used here for simplicity; actual size may vary).

---

### **Layers in a Dockerfile**

The instructions in the `Dockerfile` are a mix of **layer-creating instructions** and **non-layer-creating instructions**. These **layers** play a key role in determining the image size. If your `Dockerfile` contains multiple layer-creating instructions, it results in a **larger image size**. This is why **multi-stage builds** are often recommended—they allow the separation of layer-creating instructions into different "stages," producing more optimized and smaller images.


To deepen your understanding of multi-stage Docker builds (Day 5), you can explore the following resources:

- **YouTube Tutorial**: [Multi-Stage Docker Builds Explained](https://www.youtube.com/watch?v=8q3pJfE6Z_E&ab_channel=CloudWithVarJosh)  
- **GitHub Repository**: [Day 05: Multi-Stage Docker Builds](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2005)  



| **Layer-Creating Instructions**        | **Non-Layer-Creating Instructions**            |
|----------------------------------------|------------------------------------------------|
| `FROM`, `RUN`, `COPY`, `ADD`           | `CMD`, `ENTRYPOINT`, `WORKDIR`, `EXPOSE`, `ENV`, `LABEL`, `USER`, `VOLUME`, `STOPSIGNAL`, `ARG` |

### **Example: Layers in Our Dockerfile**

Let’s review the `Dockerfile` to identify which instructions create new layers:

- **Layer-Creating Instructions**:
  - `FROM python:3.9-slim`: Defines the base image.
  - `ADD app.py app.py`: Copies the `app.py` file into the image.
  - `RUN pip install flask`: Installs Flask (creates a new layer).
- **Non-Layer-Creating Instructions**:
  - `WORKDIR /app`: Sets the working directory.
  - `CMD ["python", "app.py"]`: Defines the default command.
  - `EXPOSE 5000`: Exposes port 5000.

---

### **How to Inspect Image Layers**

You can inspect the layers of a Docker image using the `docker image history` command. For example:
```bash
docker image history my-image
```
This will display a breakdown of the instructions and how much storage each layer-creating instruction contributes. Here’s an example output:

```plaintext
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
dbc9f6d3b551   17 hours ago   CMD ["python" "app.py"]                         0B        buildkit.dockerfile.v0
<missing>      17 hours ago   EXPOSE map[5000/tcp:{}]                         0B        buildkit.dockerfile.v0
<missing>      17 hours ago   RUN /bin/sh -c pip install flask # buildkit     11.5MB    buildkit.dockerfile.v0
<missing>      17 hours ago   ADD app.py app.py # buildkit                    170B      buildkit.dockerfile.v0
<missing>      17 hours ago   WORKDIR /app                                    0B        buildkit.dockerfile.v0
<missing>      3 months ago   FROM python:3.9-slim                            97.1MB    debuerreotype 0.15
```

---

### **How Docker Containers Use Image Layers**

When you create a container from an image:
1. **Read-Only Layers**:  
   The layers in the image are **read-only**. Each container shares the same image layers, meaning if your image is 50MB, all containers will collectively use the same 50MB of storage for those read-only layers.  
2. **Writable Layer (Delta Data)**:  
   Any new changes or files you add to the container are stored in a **writable layer** unique to that container. This is known as **delta data**.

---

### **Storage Calculation Example**

![Alt text](/images/24b.png)

Let’s assume:
- The base image size is **50MB**.
- Three containers are running from the same image:  
  - **Container 1**: Adds 4MB of new data (delta).  
  - **Container 2**: Adds 10MB of new data.  
  - **Container 3**: Adds 6MB of new data.  

**Total Storage Usage**:  
- Shared image layers: **50MB** (used by all containers).  
- Delta data: **4MB + 10MB + 6MB = 20MB**.  
- **Final Total**: **50MB + 20MB = 70MB**.

---

### **Handling Data Loss in Writable Layers**

![Alt text](/images/24c.png)

Since the image layers are **read-only**, any changes made to the container are stored in the **writable layer**. If a container is stopped or fails:
- For instance, if **Container 2** stops, the **10MB of delta data in its writable layer will be lost**.  
- This highlights the need for a mechanism to **persist container data**, ensuring important data survives beyond the container's lifecycle.

---

### **Need for Persistent Storage**
To avoid such data loss, Docker provides **volumes**—a powerful mechanism for storing data **outside the container** in a persistent manner. Volumes allow data to be accessible and survive even when containers are stopped, restarted, or deleted.

---

## **Storage in Docker: How Does It Work?**

![Alt text](/images/24d.png)

When you create a **Docker container**, it needs a way to store data. Docker achieves this with two key components:
- **Storage Drivers** 
- **Volume Drivers**

### **1. Storage Drivers**  

- **Purpose**:  
Storage drivers handle **ephemeral container data**, which exists only during the lifecycle of a container. When a container is removed, any data stored by the storage driver is lost unless persistent storage (e.g., volumes) is explicitly configured.

- **Analogy**:  
  Think of storage drivers as the **file systems for containers**, responsible for managing and organizing how container-internal storage operates.

- **Key Responsibilities**:  
  1. Manage the **container layers**, which include the base image and any subsequent changes made during runtime.
  2. Organize files and ensure efficient storage operations inside the container.

- **Kubernetes Connection**:  
  Kubernetes relies on storage drivers through container runtimes like **containerd** or **CRI-O**. These runtimes use storage drivers (e.g., OverlayFS) to handle ephemeral container data in the same way Docker does.

- **Common Examples of Docker Storage Drivers**:  
  - **OverlayFS**: The default storage driver on most modern Linux distributions. It efficiently handles layered filesystems by creating a "stack" of read-only and writable layers.  
  - **AUFS**: A legacy driver used on older systems. It has been replaced by OverlayFS due to better performance and simplicity.  
  - **ZFS**: An advanced storage driver that supports **snapshots**, **compression**, and scalable storage. Ideal for environments requiring advanced data management.  
  - **Btrfs**: A modern, high-performance storage driver that supports features like **snapshots**, **quotas**, and **subvolumes**, making it suitable for large-scale and complex storage requirements.

- **Why It Matters**:  
  Storage drivers are **crucial for handling ephemeral data** efficiently, but they are not designed to manage persistent storage. That’s why Docker and Kubernetes use **volume drivers** for data that must persist beyond the lifecycle of a container.

**Check your storage driver:**
```bash
docker info
```
Look for: `Storage Driver: overlay2`

Inside the VM, Docker typically uses the **`overlay2`** storage driver. This driver manages image layers and the writable container layer.

---

### **2. Volume Drivers**  

- **Purpose**:  
   Volume drivers manage **persistent data storage** that exists **outside the container’s lifecycle**. They allow data to survive beyond container restarts, failures, or deletions, ensuring long-term accessibility.

- **Analogy**:  
   Think of volume drivers as **external hard drives for containers**, providing a separate, reliable space to store and manage data that needs to persist independently of the containers.

- **Key Responsibilities**:  
   1. Provide a mechanism to store data **outside** the ephemeral writable layer of containers.  
   2. Allow data sharing between multiple containers, enabling collaborative or distributed workflows.  
   3. Integrate with external storage systems for advanced storage solutions (e.g., cloud-based storage or networked file systems).

- **Kubernetes Connection**:  
   - Kubernetes operates similarly but manages **Persistent Volumes (PVs)** at the **cluster level**, instead of at the container level.  
   - Kubernetes uses **Persistent Volume Claims (PVCs)** as an abstraction for applications to request storage, with backend storage provisioned by **CSI drivers**.  
   - Storage in Kubernetes integrates with volume drivers, enabling flexibility across nodes and distributed systems.

- **Common Examples of Volume Drivers**:  
   - **Local (Default Driver)**:  
      - Provides local storage on the host machine.  
      - Simple and reliable for most use cases, but limited to the host.  
   - **NFS (Network File System)**:  
      - Offers shared storage that multiple containers can access across hosts.  
      - Useful in distributed environments requiring shared data.  
   - **AWS EBS (Elastic Block Store)**:  
      - Cloud storage for containers running on AWS infrastructure.  
      - Highly scalable and integrates with other AWS services.  
   - **Azure Disks**:  
      - Microsoft’s cloud-based storage solution.  
      - Ideal for containers running in Azure environments.  

- **Why It Matters**:  
   1. Volumes decouple data from the container lifecycle, ensuring it remains intact regardless of container states.  
   2. They enable seamless integration with external storage systems, making them ideal for production and distributed applications.  
   3. Volume drivers allow data portability, scalability, and reliability, especially for large-scale, multi-container deployments.

 **Check your Volume Drivers**

- Docker uses the **`local` volume driver** by default.
- On macOS and Windows, volumes live inside the VM (not directly visible from the host).
- To share files between host and container, you can bind-mount a host directory:

  - macOS:
    ```bash
    docker run -v /Users/yourname/data:/app/data ...
    ```

  - Windows:
    ```bash
    docker run -v C:\Users\yourname\data:/app/data ...
    ```

**Inspect a volume:**
```bash
docker volume inspect <volume_name>
```

Look for: `"Driver": "local"`

---

**Note: Kubernetes Storage vs Docker Volumes**  
While Docker uses **volumes** to save data and **storage drivers** to handle temporary container files, Kubernetes takes things a step further—especially when you have many machines working together in a cluster.

In Kubernetes:
- It introduces something called **Persistent Volumes (PVs)** – think of these like shared hard drives for your apps.
- And then there are **Persistent Volume Claims (PVCs)** – your app just "asks" for the storage it needs, and Kubernetes takes care of finding it.

Behind the scenes, Kubernetes uses special plugins called **CSI (Container Storage Interface) drivers** to connect to storage from different cloud providers like **AWS EBS, Azure Disks, or NFS**.

For short-term (temporary) data, Kubernetes still uses the same type of **storage drivers** as Docker – like `overlay2`.

We’ll break all this down more clearly in the **next lecture**, where we’ll talk about how Kubernetes makes managing storage across multiple servers much easier. 

---

## **How Docker Storage Works on macOS, Windows, and Linux**

![Alt text](/images/24e.png)

Docker behaves differently depending on the operating system because containers require a Linux kernel. Here's how storage is handled across platforms:

### **Linux**

- On **Linux**, Docker runs **natively** without any virtualization layer.
- Storage and volumes are created directly on the host filesystem.
- You can access Docker volumes at:  
  ```
  /var/lib/docker/volumes
  ```

---

### **macOS and Windows**

Docker does **not** run natively on macOS or Windows. Instead, **Docker Desktop** provides a Linux environment using a lightweight virtual machine.

| OS        | Virtualization Backend                         |
|-----------|------------------------------------------------|
| macOS     | Docker Desktop uses a Linux VM via the **Apple Virtualization Framework** (previously **HyperKit**) |
| Windows   | Docker Desktop uses **WSL 2 (Windows Subsystem for Linux)** to run the Linux kernel |

**Can You Install Docker Engine Directly on macOS or Windows?**

❌ **No, not directly like on Linux.**
- Both **macOS** and **Windows** do **not have a native Linux kernel**, which Docker Engine **requires** to run containers.
- Docker Engine relies heavily on Linux features like **cgroups**, **namespaces**, and **UnionFS (e.g., overlay2)** — which are unavailable on macOS and Windows.

**NOTE**: While you can install **Docker Desktop** on a **Linux system**, it’s generally not recommended because Linux natively supports Docker Engine, which is more lightweight and efficient. When you run Docker Desktop on Linux, it still creates a lightweight virtual machine using the system's virtualization technology, adding an unnecessary layer of overhead compared to using Docker Engine directly on the host.

--- 

### **Understanding Docker Volumes**

Now that we’ve understood Docker image layers and how storage works during image creation, let’s move on to another critical aspect of Docker: **Volumes**.

In Docker, containers are ephemeral by default—this means any data created inside a container is lost when the container is deleted. To persist data beyond the container lifecycle, we use **Docker volumes**.

**Benefits of Docker Volumes**
1. **Data Persistence:**  
   Ensures data is not lost when containers restart or are removed.
   
2. **Sharing Data Between Containers:**  
   Enables multiple containers to access and modify the same data.

3. **Performance Optimization:**  
   Using external storage systems (via volume drivers) can improve performance and scalability.

---

### **Where Are Docker Volumes Stored?**

By default, Docker stores volumes inside the Docker host at:

```
/var/lib/docker/volumes
```

Every time you create a named volume, Docker creates a corresponding directory under the path above. This allows data to survive container restarts, recreation, and even deletions.

---

## **Demo: With Persistent Storage (Docker Volume)**

Now let’s do the same thing, but with a **named volume** mounted to `/app`.

> 🧪 We’ll create another container named `my-cont-2` and mount a volume to persist data.

```bash
docker volume create my-vol
docker run -d --name my-cont-1 -v my-vol:/app my-image
```

Then, exec into the container and create the file again:

```bash
docker exec -it my-cont-1 bash
cd /app
echo "This is my delta data" > file.txt
cat file.txt   # ✅ File exists
```

Now, delete the container:

```bash
docker rm -f my-cont-1
```

Even though the container is gone, the data still exists. You can verify this by checking the volume’s data directly:

```bash
sudo cat /var/lib/docker/volumes/my-vol/_data/file.txt
# Output: This is my delta data
```

**This is the power of Docker volumes**—your data is decoupled from the container’s lifecycle.

---

### **Note for macOS and Windows Users**

On **Linux**, Docker runs natively and stores volumes under:

```
/var/lib/docker/volumes
```

However, on **macOS** (and Windows), Docker runs inside a lightweight **virtual machine (VM)** using a technology like **HyperKit** or **WSL2**, depending on the system.

As a result, you **won’t find** the `/var/lib/docker/volumes` directory on your local Mac filesystem — it exists **inside the Docker VM**.

#### How to Access Volume Data on macOS:

If you're on macOS/Windows and want to inspect volume data:

1. Use Docker commands to access it via the container:
   ```bash
   docker run -it --rm -v my-vol:/app alpine sh
   cd /app
   ls -l
   ```

2. Or use `docker cp` to copy data from a container to your local filesystem:
   ```bash
   docker cp my-cont-2:/app/file.txt .
   ```

---

### **Types of Docker Volumes: Which to Use When?**

Docker supports different volume types, and each has its own use case:

| **Type**            | **Use Case**                                                                 | **Pros**                                               | **Cons**                                             |
|---------------------|------------------------------------------------------------------------------|---------------------------------------------------------|------------------------------------------------------|
| **Named Volume**    | Default Docker-managed volumes (e.g., `my-vol`)                              | Easy to use, portable, survives container deletion      | Abstracted path, may need extra step for backups     |
| **Bind Mount**      | Mount a specific host directory to the container                             | Full control, real-time syncing, easy for dev work      | Tied to host path, can cause permission issues       |
| **tmpfs Mount**     | Temporary in-memory mount (data lost on restart)                             | Fast, doesn't write to disk                             | Data not persistent, only useful for short-term use  |

In most production use-cases, **named volumes** are preferred because they are **managed, portable, and persistent**.  
For development and local testing, **bind mounts** are useful when you want to sync code from your local machine into the container.

---

### **Conclusion**

Docker volumes are essential for managing **persistent storage** in containerized environments. While containers are designed to be ephemeral, volumes provide a way to preserve data even if the container is stopped, removed, or recreated.

We also saw how the behavior of storage slightly differs across **Linux**, **macOS**, and **Windows** due to the use of virtualization layers like **Docker Desktop** (with **HyperKit** on macOS and **WSL2** on Windows). Regardless of platform, understanding how Docker handles data—through **storage drivers**, **volume drivers**, and volume mounting—lays the foundation for more complex systems like **Kubernetes storage** with **Persistent Volumes** and **CSI drivers**.

In the next lecture, we’ll extend this understanding to the Kubernetes world and learn how storage is handled across a cluster!

---

# **Kubernetes Core and Extended Architecture**  

![Alt text](/images/25a.png)

Kubernetes is a powerful container orchestration platform built on a **modular design**. This modularity enables **flexibility, scalability, and extensibility**. The **core components** handle essential orchestration tasks, while **plugins, add-ons, and third-party extensions** extend functionality without bloating the core system.  

This section clarifies the distinction between **Kubernetes Core** and **External Extensions**, explaining how various components interact within the overall architecture.  

---

## **Kubernetes Core Architecture**  

### **What is Kubernetes Core?**  
The **Kubernetes Core** consists of essential built-in components responsible for cluster management and orchestration. These components are categorized into **Control Plane Components** and **Node Components**.  

### **1️⃣ Control Plane Components**  
The control plane manages cluster operations, making scheduling decisions and maintaining the desired state.  

- **API Server:** The central control plane component that validates and processes REST API requests, enforcing security policies and forwarding requests to relevant components.  
- **Scheduler:** Assigns Pods to nodes based on resource availability and scheduling policies.  
- **Controller Manager:** Runs control loops to maintain the desired state (e.g., replication, endpoint management).  
- **etcd:** A distributed, consistent key-value store that persistently stores all cluster state and configuration data.  
- **Cloud Controller Manager:** Manages cloud-provider-specific integrations such as external load balancers, persistent storage provisioning, and node lifecycle management.  

### **2️⃣ Node Components**  
Nodes are the worker machines that run containerized workloads.  

- **kubelet:** The primary node agent that ensures containers are running as expected.  
- **kube-proxy:** Maintains network rules on each node, enabling seamless service discovery and communication between Pods.  

---

## **Kubernetes Extensions: Beyond the Core**  
While the **Kubernetes Core** provides essential orchestration functionalities, additional features like networking, storage, monitoring, and security are implemented via **external components**.  

### **1️⃣ Plugins**  

![Alt text](/images/25b.png)

Plugins extend Kubernetes by enabling external integration while adhering to standardized APIs. 


CNI (Container Network Interface), CRI (Container Runtime Interface), and CSI (Container Storage Interface) are standardized interfaces that define how networking, runtime, and storage components interoperate with Kubernetes. These interfaces adhere to specifications that are developed and maintained by the Kubernetes community under the governance of the CNCF. Although the CNCF endorses these standards, it is the community that actively defines and updates the specifications.

Anyone can develop their own plugins for CNI, CRI, and CSI as long as they conform to these specifications, ensuring compatibility and interoperability within Kubernetes environments.


- **Container Network Interface (CNI):** Configures Pod networking and IP allocation.  
- **Container Storage Interface (CSI):** Manages external storage solutions.  
- **Container Runtime Interface (CRI):** Allows Kubernetes to interact with various container runtimes (e.g., containerd, CRI-O).  
 

#### **CRI Plugins (Container Runtimes)**
*(The following list is indicative, not exhaustive.)*
| Plugin        | Description |
|--------------|------------|
| **containerd** | Default for most Kubernetes setups; lightweight, optimized, and follows OCI standards. |
| **CRI-O** | Minimal runtime designed for Kubernetes; integrates tightly with OpenShift. |
| **Kata Containers** | Uses lightweight VMs for security; ideal for isolating untrusted workloads. |
| **gVisor (by Google)** | User-space sandboxing for enhanced security; limits direct host access. |

#### **CNI Plugins (Networking)**
*(The following list is indicative, not exhaustive.)*
| Plugin        | Description |
|--------------|------------|
| **Calico** | Supports network policies and BGP routing; ideal for security-focused environments. |
| **Cilium** | High-performance with eBPF-based networking; excels in observability and security. |
| **Flannel** | Lightweight and simple; lacks network policy support. |
| **Weave Net** | Provides encrypted pod-to-pod communication; supports multi-cluster setups. | 

#### **CSI Plugins (Storage)**
*(The following list is indicative, not exhaustive.)*
| Plugin        | Description |
|--------------|------------|
| **Amazon EBS CSI** | Provides block storage for Kubernetes workloads on AWS; supports dynamic provisioning. |
| **Azure Disk CSI** | Offers high-performance, persistent storage for Kubernetes on Azure. |
| **Google PD CSI** | Integrates with Google Cloud Persistent Disks; supports snapshots and resizing. |
| **Ceph RBD CSI** | Ideal for scalable, distributed storage; supports snapshots and cloning. |
| **Portworx CSI** | Enterprise-grade storage with high availability, backups, and replication. |
| **OpenEBS CSI** | Lightweight and cloud-native storage for Kubernetes; optimized for local PVs. |
| **Longhorn CSI** | Rancher’s distributed block storage for Kubernetes; supports snapshots and disaster recovery. |

#### **Responsibilities of CRI, CNI, and CSI in Kubernetes**  
*(The responsibilities listed below are indicative, not exhaustive.)*  

**CRI (Container Runtime Interface) - Manages Container Lifecycle**  
- Pulls container images from registries.  
- Starts, stops, and deletes containers.  
- Manages container networking via CNI integration.  
- Handles logging and monitoring of containers.  
- Allocates resources like CPU and memory to containers.  
- Ensures compatibility with Kubernetes via standardized gRPC APIs.  

**CNI (Container Network Interface) - Manages Pod Networking**  
- Assigns IP addresses to pods.  
- Enables communication between pods across nodes.  
- Manages network policies for security and isolation.  
- Supports service discovery and DNS resolution.  
- Integrates with cloud and on-prem networking solutions.  
- Provides network metrics and observability features.  

**CSI (Container Storage Interface) - Manages Storage for Pods**  
- Dynamically provisions persistent storage for workloads.  
- Supports mounting and unmounting of storage volumes.  
- Handles volume resizing, snapshots, and backups.  
- Ensures data persistence across pod restarts.  
- Integrates with various cloud and on-prem storage providers.  
- Manages access controls and multi-node storage sharing.  


### **2️⃣ Add-Ons**  
Add-ons enhance Kubernetes with additional functionalities:  

- **Ingress Controllers:** Handle external access, load balancing, and routing.  
- **Monitoring Tools:** Observability tools like Prometheus and Grafana.  
- **Service Meshes:** Solutions like Istio and Linkerd enhance inter-service communication.  

### **3️⃣ Third-Party Extensions**  
Third-party tools and solutions integrate seamlessly into Kubernetes:  

- **Helm:** Kubernetes package manager for simplified deployment.  
- **Prometheus:** Monitoring and alerting system for cloud-native applications.  
- **Istio:** Service mesh for security, traffic management, and observability.  

---

## **Deep Dive into Kubernetes Interfaces**  
Kubernetes uses standardized interfaces for seamless extensibility. The three primary interfaces are **CSI, CNI, and CRI**, each designed to manage storage, networking, and runtime integration.  

### **Overview of Kubernetes Interfaces**  

| **Interface** | **Purpose** | **Key Features** | **Common Implementations** |  
|--------------|------------|------------------|---------------------------|  
| **Container Storage Interface (CSI)** | Enables external storage integration without modifying Kubernetes core. | - Decouples storage drivers from Kubernetes. <br> - Supports snapshots, volume expansion, and lifecycle management. <br> - Uses gRPC-based APIs (`CreateVolume`, `DeleteVolume`, etc.). | - AWS EBS CSI Driver <br> - GCE Persistent Disk CSI Driver <br> - Ceph RBD CSI Driver |  
| **Container Network Interface (CNI)** | Standardizes how Kubernetes manages networking. | - Configures Pod networking and IP allocation. <br> - Supports network policies and encryption. <br> - Enables scalable and secure communication. | - Calico (Network policies) <br> - Cilium (eBPF-based performance) <br> - AWS VPC CNI (AWS-native networking) <br> - Weave Net (Encrypted multi-cluster networking) |  
| **Container Runtime Interface (CRI)** | Defines how Kubernetes interacts with different container runtimes. | - Manages container lifecycle (image pulling, creation, execution, logging). <br> - Ensures runtime consistency. <br> - Enables Kubernetes to work with multiple runtimes. | - containerd (Lightweight, Kubernetes-native) <br> - CRI-O (Minimal runtime for Kubernetes) <br> - Podman (Daemonless alternative) <br> - Docker (Previously supported via dockershim; now deprecated) |  

---

## **Why Kubernetes Uses a Plugin-Based Architecture?**  

Kubernetes’ **plugin-based architecture** is fundamental to its flexibility, scalability, and vendor neutrality. Here’s why:  

- **Interoperability:** Kubernetes interacts with various **networking, storage, and runtime solutions** without tight integration into its core.  
- **Flexibility & Vendor Neutrality:** Users can select the **best tools for their workloads** without being locked into a specific vendor.  
- **Encourages Innovation:** Plugin developers can **iterate and improve** their solutions independently, without requiring changes to Kubernetes itself.  
- **Scalability & Maintainability:** Individual components can be **scaled and upgraded independently**, improving reliability and maintainability.  

For example, **Cilium**, a CNI plugin using **eBPF**, enhances network security and observability without requiring modifications to Kubernetes' core networking model. This level of extensibility allows Kubernetes to adapt seamlessly across different infrastructure environments.  

---

## **Key Takeaways**  

- **Kubernetes Core** consists of essential control plane and node components (API Server, Scheduler, Controller Manager, etc.).  
- **Plugins** (CNI, CSI, CRI) and **Add-ons** (Ingress, Monitoring, Service Mesh) extend Kubernetes functionality beyond the core.  
- **CSI** allows for **scalable and flexible storage integrations** while adhering to strict API compliance.  
- **Modular architecture** ensures vendor neutrality, encourages innovation, and simplifies upgrades.  
- Kubernetes’ **plugin ecosystem** plays a crucial role in its extensibility, allowing it to adapt to diverse infrastructure needs.  

---

### **Ephemeral Storage**

Ephemeral storage refers to storage that is temporary, meaning that any data written to it only lasts for the duration of the Pod’s lifetime. When the Pod is deleted, the data is also lost.

#### **EmptyDir**

![Alt text](/images/26a.png)

**What is EmptyDir?**  
  - `emptyDir` is an empty directory created when a Pod is assigned to a node. The directory exists for the lifetime of the Pod.
  - If you define an `emptyDir` volume in a Deployment, each Pod created by the Deployment will have its **own unique** `emptyDir` volume. Since `emptyDir` exists **only as long as the Pod is running**, any data stored in it is **lost when the Pod is terminated or deleted**.  
  - Defining an `emptyDir` in a Deployment ensures that **each Pod replica gets its own isolated temporary storage**, independent of other replicas. This is particularly useful for workloads where each Pod requires **scratch storage** that doesn’t need to persist beyond the Pod’s lifecycle.
  
**Why is it Used?**  
  It is ideal for scratch space (Temporary workspace for processing.), caching, or temporary computations where data persistence isn’t required.
  
Sure! Here's a refined and enhanced version of your message that improves clarity, grammar, and flow, while keeping the technical concepts intact:

---

**Writable vs Read-Only Layers in Containers (Day 24 Recap)**

In [Day 24](https://www.youtube.com/watch?v=5t3yHFoqK6g&t=12s&ab_channel=CloudWithVarJosh), we explored how **container images** work under the hood—specifically the concept of **read-only and writable layers**.

Let’s break it down:

If a Pod has **three containers** that all use the **same container image**, they **share the read-only image layer** to save space and optimize resource usage. However, **each container still gets its own writable layer**. So, any file changes or modifications done by one container remain **isolated** and are **not visible to the others**.

But what if you want them to share data?

That’s where **volumes** come into play—specifically, an `emptyDir` volume. If you mount the **same `emptyDir` volume** into all three containers, they now have access to a **shared writable space**. This allows them to **collaborate and share data** during the Pod’s lifetime, effectively **creating a shared writable layer** across containers.


Check out the **Day 24** content below:
**GitHub:** [Day 24 GitHub Notes](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2024)    
**YouTube:** [Day 24 YouTube Video](https://www.youtube.com/watch?v=5t3yHFoqK6g&t=12s&ab_channel=CloudWithVarJosh)

---

#### Demo: emptyDir  
  ```yaml
  apiVersion: v1  # Specifies the API version used to create the Pod.
  kind: Pod       # Declares the resource type as a Pod.
  metadata:
    name: emptydir-example  # Name of the Pod.
  spec:
    containers:
    - name: busybox-container  # Name of the container inside the Pod.
      image: busybox           # Using the lightweight BusyBox image.
      command: ["/bin/sh", "-c", "sleep 3600"]  # Overrides the default command. Keeps the container running for 1 hour (3600 seconds).
      volumeMounts:
      - mountPath: /data       # Mount point inside the container where the volume will be accessible.
        name: temp-storage     # Refers to the volume defined in the `volumes` section below.
    - name: busybox-container-2  # Name of the container inside the Pod.
      image: busybox           # Using the lightweight BusyBox image.
      command: ["/bin/sh", "-c", "sleep 3600"]
      volumeMounts:
      - mountPath: /data
        name: temp-storage
    volumes:
    - name: temp-storage       # Name of the volume, must match the name in `volumeMounts`.
      emptyDir: {}             # Creates a temporary directory that lives as long as the Pod exists.
                              # Useful for storing transient data that doesn't need to persist.

  ```
  *Here, `/data` is an emptyDir volume that will be removed along with the Pod.*


---

**Verification**

We will now verify that both containers within the same Pod can access the shared volume mounted using `emptyDir`.

**Step 1: Create a file from `busybox-container`**

```bash
kubectl exec emptydir-example -c busybox-container -- sh -c 'echo "What a file!" > /data/myfile.txt'
```

This command writes the string `"What a file!"` to a file named `myfile.txt` inside the shared `/data` directory.

**Step 2: Read the file from `busybox-container-2`**

```bash
kubectl exec emptydir-example -c busybox-container-2 -- cat /data/myfile.txt
```

**Expected Output:**

```
What a file!
```

---

**Key Takeaway:**

This demonstrates that both containers within the same Pod can access and share the same volume mounted using `emptyDir`.  
This level of sharing is **not possible with the Pod's default ephemeral storage**, as each container has its own isolated writable layer.  
Using `emptyDir` enables inter-container file sharing within a Pod.

---

#### **Downward API**

![Alt text](/images/26a.png)

**What is the Downward API?**  
The Downward API in Kubernetes provides a mechanism for Pods to access metadata about themselves or their environment. This metadata can include information such as the Pod's name, namespace, labels, annotations, or resource limits. It is injected into containers in a Pod either as **environment variables** or through mounted **files via volumes**.

When the Downward API is configured in a Deployment, each Pod created by the Deployment gets its own unique set of metadata based on the Pod's attributes. This allows Pods to retrieve runtime-specific details dynamically, without hardcoding or manual intervention.

#### **Why is it Used?**  
- **Dynamic Configuration**: Enables applications to dynamically retrieve Pod-specific metadata, such as labels or resource limits.  
- **Self-Awareness**: Makes Pods aware of their environment, including their name, namespace, and resource constraints.  
- **Simplifies Configuration Management**: Helps eliminate the need for manual configuration by providing metadata directly to the containers.

---

**💡 Did You Know?**

Back in [Day 21](https://www.youtube.com/watch?v=VEwP_wF67Tw&ab_channel=CloudWithVarJosh), we discussed the **sidecar pattern**—a powerful design pattern in Kubernetes often used for logging, monitoring, and proxy use cases. These **sidecar (helper) containers** frequently rely on the **Downward API** to access real-time metadata such as the Pod name, namespace, labels, and resource limits.

Without the Downward API, these sidecars would need to **continuously poll the Kubernetes API server** to fetch this metadata, increasing API server load and introducing unnecessary network overhead. By using the Downward API, they can access this data **locally within the Pod**, improving performance and **offloading the API server**.

For example, imagine you're running a monitoring agent as a sidecar, and you want to collect metrics or logs for all Pods within a specific namespace like `app1-ns`. If the agent doesn’t know **which Pod it's running in** or **which namespace it belongs to**, it wouldn't be able to label or filter that data correctly. The Downward API solves this problem by **injecting runtime-specific metadata directly into the container**, making it **self-aware**.

👉 Explore **Day 21 (Sidecar Pattern)**:  
**GitHub:** [Day 21 GitHub Notes](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2021)
**YouTube:** [Day 21 YouTube Video](https://www.youtube.com/watch?v=VEwP_wF67Tw&ab_channel=CloudWithVarJosh)

---

#### Demo: downwardAPI - Environment variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: downwardapi-example
  labels:
    app: demo
spec:
  containers:
  - name: metadata-container
    image: busybox
    command: ["/bin/sh", "-c", "env && sleep 3600"] 
    # The container prints all environment variables and then sleeps for 1 hour.
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
      # Creates an environment variable named POD_NAME.
      # The value of this variable is set dynamically using the Downward API.
      # It pulls the Pod's name (in this case, "downwardapi-example") from its metadata.
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
      # Creates an environment variable named POD_NAMESPACE.
      # The value of this variable is set dynamically using the Downward API.
      # It pulls the Pod's namespace (e.g., "default") from its metadata.
```

**Verification**


Run the following `kubectl exec` command to execute `env` inside the container:

```bash
kubectl exec downwardapi-example -- env | grep POD_
```

You should see output like:

```
POD_NAME=downwardapi-example
POD_NAMESPACE=default
```

This confirms that:
- `POD_NAME` is set to the Pod’s name.
- `POD_NAMESPACE` is set to the namespace it's running in (usually `default`, unless specified otherwise).

---

#### Demo: downwardAPI - Files via Volumes

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: downwardapi-volume
  labels:
    app: good_app
    owner: hr
  annotations:
    version: "good_version"
spec:
  containers:
  - name: metadata-container
    image: busybox
    command: ["/bin/sh", "-c", "cat /etc/podinfo/* && sleep 3600"]
    # The container will display the contents of all files under /etc/podinfo (i.e., metadata)
    # and then sleep for an hour, keeping the pod alive for verification.
    volumeMounts:
    - name: downwardapi-volume
      mountPath: /etc/podinfo
      # Mounts the downward API volume at /etc/podinfo inside the container.
  volumes:
  - name: downwardapi-volume
    downwardAPI:
      items:
      - path: "labels"
        fieldRef:
          fieldPath: metadata.labels
        # Writes the Pod's labels to a file named 'labels' under /etc/podinfo.
      - path: "annotations"
        fieldRef:
          fieldPath: metadata.annotations
        # Writes the Pod's annotations to a file named 'annotations' under /etc/podinfo.

```

**Verification**

**Step 1: Exec into the Pod**

```bash
kubectl exec -it downwardapi-volume -- /bin/sh
```

Once inside the container shell, run:

```bash
cd /etc/podinfo
ls -l
```

You should see symbolic links:

```
annotations -> ..data/annotations
labels -> ..data/labels
```

These links are created because Kubernetes uses a [projected volume](https://kubernetes.io/docs/concepts/storage/volumes/#projected) with the Downward API, which manages file updates using symlinks pointing to the `..data/` directory. This allows for atomic updates.

---

**Step 2: View the Contents**

Check the content of each file:

```bash
cat labels
```

Expected output:
```
app="good_app"
owner="hr"
```

```bash
cat annotations
```

Expected output:
```
version="good_version"
```

These values are directly fetched from the pod’s metadata and written to files using the Downward API.

---

**Step 3: Verify Using Pod Logs**

Since the pod's container command was set to:

```yaml
command: ["/bin/sh", "-c", "cat /etc/podinfo/* && sleep 3600"]
```

The contents of `/etc/podinfo/labels` and `/etc/podinfo/annotations` will be printed in the pod's logs when it starts. To view them:

```bash
kubectl logs downwardapi-volume
```

Expected output:
```
app="good_app"
owner="hr"
version="good_version"
```

This further confirms that the Downward API volume successfully mounted the metadata into the container at runtime.


---

#### **ConfigMaps & Secrets (To Be Discussed in Next Lecture)**

- **Overview:**  
  ConfigMaps and Secrets are Kubernetes objects designed to inject configuration data and sensitive information into Pods. While they exist independently of Pods, when mounted as volumes, the data is temporary and only available for the Pod’s lifetime.
  
- **Why are they Used?**  
  They allow the decoupling of configuration from application code and the secure injection of sensitive data, simplifying management across multiple Pods.


---

## Evolution of Storage in Kubernetes: From In-Tree to CSI Drivers

![Alt text](/images/27a.png)

Managing **persistent storage** in Kubernetes has come a long way. Initially, storage drivers were built directly into Kubernetes' core codebase, known as **in-tree volume plugins**. Over time, this tightly coupled model proved limiting, leading to the adoption of the **Container Storage Interface (CSI)**—a modular and extensible solution for storage integration.

This write-up explores the transition from in-tree plugins to CSI drivers and explains where commonly-used storage types fit into Kubernetes today.

---

## 1. In-Tree Volume Plugins: The Legacy Model

In-tree volume plugins were integrated directly into Kubernetes' codebase. Adding or updating these plugins required modifying Kubernetes itself, resulting in several drawbacks:

- Maintenance was cumbersome and tied to Kubernetes release cycles.
- Vendors could not independently develop or release updates.
- Bugs in storage drivers could affect Kubernetes core functionality.

### Examples of In-Tree Plugins:
- `awsElasticBlockStore` (AWS EBS)
- `azureDisk`, `azureFile`
- `gcePersistentDisk` (GCE PD)
- `nfs`, `glusterfs`
- `rbd` (Ceph RADOS Block Device)
- `iscsi`, `fc` (Fibre Channel)

**Note:** Most in-tree plugins are deprecated and replaced with CSI drivers.

---

## 2. Container Storage Interface (CSI): The Modern Standard

To address the limitations of in-tree plugins, Kubernetes adopted the **Container Storage Interface (CSI)**, developed by the **Cloud Native Computing Foundation (CNCF)**. CSI decouples storage driver development from Kubernetes core, allowing vendors to independently create and maintain their drivers.

### Key Benefits of CSI:
- **Independent updates:** Vendors can release drivers without waiting for Kubernetes updates.
- **Faster development:** Features and fixes are delivered more quickly.
- **Flexibility:** CSI supports advanced capabilities like snapshotting, resizing, cloning, and monitoring.
- **Easy integration:** Drivers for custom use cases can be developed with ease.

### Examples of CSI Drivers:
- AWS EBS CSI Driver: `ebs.csi.aws.com`
- GCE PD CSI Driver: `pd.csi.storage.gke.io`
- Azure Disk CSI Driver: `disk.csi.azure.com`
- Ceph CSI Driver
- OpenEBS CSI Drivers (e.g., for cStor, Jiva, etc.)
- Portworx CSI, vSphere CSI

**Note:** Starting with Kubernetes 1.21, most in-tree volume plugins were officially deprecated, and CSI became the default for handling persistent volumes.

---

## Special Cases: `hostPath` and `local` Storage

### `hostPath`
- Mounts a directory from the node directly into the pod.
- Primarily used for testing or simple workloads.
- In-tree only; there is no CSI implementation.
- **Caution:** Unsuitable for production due to security risks and lack of scheduling guarantees.

### `local` Volumes
- Mounts a physical disk attached to a node.
- Can be implemented as either in-tree or CSI-based, depending on the setup.
- Useful for tightly coupled environments like on-premises clusters.
- Works well with **PersistentVolume** and supports **node affinity** for proper pod scheduling.

---

#### **Transitioning from In-Tree Drivers**
While in-tree drivers served their purpose in the early stages of Kubernetes, they are now being deprecated. CSI plugins are the recommended approach for modern storage management. For example:

**In-Tree Driver Example**:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: in-tree-pv
  # Name of the PersistentVolume resource.
spec:
  capacity:
    storage: 10Gi
    # Defines the size of the volume. This PV provides 10Gi of storage.
  accessModes:
    - ReadWriteOnce
    # The volume can be mounted as read-write by a single node.
  awsElasticBlockStore:
    # This is the legacy in-tree volume plugin for AWS EBS.
    # It allows mounting an existing AWS EBS volume into a pod.
    volumeID: vol-0abcd1234efgh5678
    # The unique ID of the EBS volume to be mounted.
    fsType: ext4
    # The filesystem type of the volume. This must match the filesystem on the EBS volume.

```

**CSI Driver Example**:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: csi-pv
  # The name of the PersistentVolume resource.
spec:
  capacity:
    storage: 10Gi
    # Defines the storage size of the volume. This volume offers 10Gi of space.
  accessModes:
    - ReadWriteOnce
    # The volume can be mounted as read-write by only one node at a time.
  storageClassName: gp3-sc
  # The StorageClass to which this volume belongs.
  # This allows dynamic provisioning and matching with a PVC that requests 'gp3-sc'.
  csi:
    # This specifies the use of a Container Storage Interface (CSI) driver,
    # which is the recommended way to provision external storage in modern Kubernetes clusters.
    driver: ebs.csi.aws.com
    # The name of the CSI driver used for AWS EBS volumes.
    volumeHandle: vol-0abcd1234efgh5678
    # The unique ID of the EBS volume in AWS.
    # This is used by the driver to locate and attach the correct volume.
    fsType: ext4
    # The filesystem type to mount. This must match the actual filesystem on the EBS volume.

```

---

#### **Key Takeaways:**
1. **In-Tree Drivers**: These legacy solutions are tightly coupled with Kubernetes and being phased out.  
2. **CSI Plugins**: The preferred method for storage integration, offering scalability and extensibility.  
3. **Dynamic Provisioning**: CSI supports automated volume creation and lifecycle management via StorageClasses.  
4. **File, Block, and Object Storage**:
   - **File and Block Storage**: Mountable as volumes in Pods using CSI plugins.  
   - **Object Storage**: Not natively mountable; access via application-level SDKs or APIs is recommended.

---

Kubernetes supports **CSI (Container Storage Interface) plugins** for both **file (NFS)** and **block storage** options. These plugins are provided by storage vendors such as **NetApp**, **Dell EMC**, and cloud providers like **AWS**, **Azure**, and **Google Cloud Platform (GCP)**. CSI plugins enable Kubernetes Pods to access file and block storage seamlessly, offering advanced features like dynamic provisioning, snapshots, and volume resizing.

For object storage solutions like **Amazon S3**, **Azure Blob Storage**, or **Google Cloud Storage (GCS)**, Kubernetes does not natively support mounting object storage as volumes because object storage operates differently from file systems. Instead, it is **recommended to use application-specific SDKs** or APIs to interact with object storage. These SDKs allow applications to efficiently access and manage object storage for tasks such as retrieving files, uploading data, and performing operations like versioning or replication.

The distinction lies in the use case:  
- **File and Block Storage**: Designed for direct mounting and integration with Kubernetes workloads using CSI plugins.  
- **Object Storage**: Better suited for application-level access via APIs or SDKs, as it was not designed to be mounted like file systems.  

By leveraging CSI plugins for file and block storage, and SDKs for object storage, you can make the most of modern, scalable storage options tailored to your Kubernetes workloads.

> Earlier, Kubernetes supported a lot of built-in drivers, called in-tree plugins. But today, the ecosystem has fully moved to **CSI drivers**, which are the future-proof way to provision and consume persistent storage. So whatever storage you’re dealing with — AWS EBS, NFS, iSCSI — if you’re doing it the modern way, you’re doing it with a **CSI driver**.

---

### **Persistent Storage**

Before diving into Persistent Volumes (PVs) and Persistent Volume Claims (PVCs), let's first discuss **hostPath**, a persistent storage option in Kubernetes. While Kubernetes does not generally recommend using `hostPath` due to its limitations and security concerns, it can still be useful in specific scenarios.

#### **hostPath**

- **What is hostPath?**  
  A `hostPath` volume mounts files or directories from the host node’s filesystem directly into the Pod.
  `hostPath` volumes are **node-specific**, meaning they are limited to the node where the Pod is scheduled. All Pods running on the same node can access and share data stored in a `hostPath` volume. However, since each node has its own independent storage, **Pods on different nodes cannot share the same `hostPath` volume**, making it unsuitable for distributed workloads requiring cross-node data sharing.
  
- **Why is it Used?**  
  It is used for scenarios such as debugging, accessing host-level files (logs, configuration files), or sharing specific host resources with containers.

**Why Kubernetes Recommends Avoiding `hostPath`**  
- **Security Risks**: Pods can access sensitive host files if `hostPath` is misconfigured.  
- **Limited Portability**: Since `hostPath` is tied to a node, it reduces flexibility in scheduling.  
- **Better Alternatives**: Kubernetes recommends using **PersistentVolumes (PV) with PersistentVolumeClaims (PVC)** or **local PersistentVolumes**, which offer better control and security. We will discuss these later in this lecture.

Learn more: [Kubernetes Documentation on hostPath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath).

> With `emptyDir`, all containers **within a single pod** can access the shared volume, but it is **not accessible to other pods**.  
> With `hostPath`, **any pod running on the same node** that mounts the same host directory path can access the same volume data, thus enabling **cross-pod sharing on the same node**.

---

### **Understanding `hostPath` in KIND: How Storage Works Under the Hood**

![Alt text](/images/27b.png)

When using KIND (Kubernetes IN Docker), it’s important to understand where your `hostPath` volumes are actually created — especially because Docker behaves differently across operating systems.

#### **1. On Ubuntu/Linux**

- On Linux distributions like **Ubuntu**, **Docker Engine runs natively** on the host OS.
- So when you define a `hostPath` volume, like `/tmp/hostfile`, it points directly to the **host’s actual filesystem** (i.e., your Ubuntu machine).
- The path `/tmp/hostfile` truly exists on the Ubuntu host, and the container will mount that exact path into the Pod.

#### **2. On macOS and Windows**

- Docker Engine **does not run natively** on macOS or Windows, since it requires a Linux kernel.
- Docker Desktop creates a **lightweight Linux VM** internally (via **HyperKit** on macOS or **WSL2** on Windows) to run the Docker Engine.
- KIND then runs each Kubernetes node (`control-plane`, `worker-1`, `worker-2`) as a **Docker container** inside that Linux VM.
- When you define a `hostPath` in a Pod spec (e.g., `/tmp/hostfile`), it **does not point to your macOS or Windows host filesystem**.
- Instead, it points to the **filesystem of the specific Docker container (the Kubernetes node) running that Pod**.
  
  > So, technically, the `hostPath` volume is **inside the container** representing your worker node — **not** on your macOS/Windows host, and **not even directly inside the lightweight Linux VM** used by Docker Desktop.

---

### **Key Takeaway**

| Platform      | Docker Engine Runs On | `hostPath` Points To                            |
|---------------|------------------------|-------------------------------------------------|
| Ubuntu/Linux  | Natively               | Host's actual filesystem (e.g., `/tmp/hostfile`) |
| macOS/Windows | Linux VM via Docker    | Filesystem **inside the Kubernetes node container** |

---

### **Why This Matters**

When testing `hostPath` on macOS/Windows using KIND, any file you write via the volume:
- Exists **only inside the worker node container**.
- Is **not visible** on your macOS/Windows host filesystem.
- Will be lost if the KIND cluster or node container is destroyed.

This is important when you're simulating persistent storage, as `hostPath` is not portable across nodes and shouldn't be used in production — but is often used for demos or local testing.

---

### Demo: hostPath

In this demonstration, we showcase how to use the `hostPath` volume type in Kubernetes to mount a file from the host node into a container.

We begin by creating the following Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-example
spec:
  containers:
  - name: busybox-container
    image: busybox
    command: ["sh", "-c", "cat /data && sleep 3600"]
    # The container reads and prints the content of /data, then sleeps for 1 hour.
    volumeMounts:
    - mountPath: /data
      name: host-volume
      # Mounts the 'host-volume' to the /data path inside the container.
      # This gives the container access to the file on the host.
  volumes:
  - name: host-volume
    hostPath:
      path: /tmp/hostfile
      type: FileOrCreate
      # If /tmp/hostfile doesn't exist on the host, it is created as an empty file before being mounted.
```

> With this configuration, the file `/tmp/hostfile` on the host becomes accessible inside the container at `/data`.

Now, let’s populate the file with content using the command below:

```bash
kubectl exec -it hostpath-example -- sh -c 'echo "Hey you!!" > /data'
```

**Understanding `hostPath` Types**

Kubernetes supports several `hostPath` volume types that define how paths on the host are managed. For detailed information on supported types such as `Directory`, `File`, `Socket`, and `BlockDevice`, refer to the official documentation:

[Kubernetes Documentation - hostPath Volume Types](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath-volume-types)

---

**Verification Across Pods on the Same Node**

Since `hostPath` uses the underlying node’s filesystem, the data can be shared between different pods **only if they are scheduled on the same node**.

To determine where the initial pod was scheduled:

```bash
kubectl get pods -o wide
```

Assuming the pod was created on a node named `my-second-cluster-worker2`, we can now schedule a new pod on the same node to validate data sharing:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-verify
spec:
  nodeName: my-second-cluster-worker2
  containers:
  - name: busybox-container
    image: busybox
    command: ["sh", "-c", "cat /data && sleep 3600"]
    volumeMounts:
    - mountPath: /data
      name: host-volume
  volumes:
  - name: host-volume
    hostPath:
      path: /tmp/hostfile
      type: FileOrCreate
```

To verify that the new pod can access the same file content:

```bash
kubectl exec hostpath-verify -- cat /data
```

You should see the output:
```
Hey you!!
```

This confirms that both pods are using the same file from the host node via `hostPath`.

---

### Persistent Volumes (PVs) & Persistent Volume Claims (PVCs)

![Alt text](/images/27c.png)
#### **Persistent Volumes (PVs)**

- **What is a PV?**  
  A PV is a piece of storage in your cluster that has been provisioned either manually by an administrator or dynamically using Storage Classes.
  
- **Key Characteristics:**  
  PVs exist independently of Pod lifecycles and can be reused or retained even after the Pod is deleted. They have properties such as **capacity, access modes, and reclaim policies**.

#### **Persistent Volume Claims (PVCs)**

- **What is a PVC?**  
  A PVC is a request for storage by a user. It functions similarly to how a Pod requests compute resources. When a PVC is created, Kubernetes searches for a PV that meets the claim's requirements.
  
- **Binding Process:**  
  1. **Administrator:** Provisions PVs (or sets up Storage Classes for dynamic provisioning).  
  2. **Developer:** Creates a PVC in the Pod specification requesting specific storage attributes.  
  3. **Kubernetes:** Binds the PVC to a suitable PV, thereby making the storage available to the Pod.


**Pods rely on Node resources—such as CPU, memory, and network—to run containers.** On the other hand, when a Pod requires **persistent storage**, it uses a **PersistentVolumeClaim (PVC)** to request storage from a **PersistentVolume (PV)**, which serves as the **actual storage backend**. This separation of compute and storage allows Kubernetes to manage them independently, improving flexibility and scalability.

---

### **Understanding Scope & Relationships of PV and PVC in Kubernetes**

In Kubernetes, **PersistentVolumes (PVs)** and **PersistentVolumeClaims (PVCs)** play a central role in persistent storage — but they differ in how they're scoped and used.

![Alt text](/images/27e.png)

#### **PVs are Cluster-Scoped Resources**
- A **PersistentVolume (PV)** is a **cluster-wide resource**, just like Nodes or StorageClasses.
- This means it is **not tied to any specific namespace**, and it can be viewed or managed from anywhere within the cluster.
- You can verify this using:
  ```bash
  kubectl api-resources | grep persistentvolume
  ```
  This shows that the resource `persistentvolumes` has **no namespace**, indicating it's **cluster-scoped**.

#### **PVCs are Namespace-Scoped**
- A **PersistentVolumeClaim (PVC)**, on the other hand, is a **namespaced resource**, just like Pods or Deployments.
- This means it exists **within a specific namespace** and is only accessible by workloads (Pods) within that namespace.
- You can verify this using:
  ```bash
  kubectl api-resources | grep persistentvolumeclaim
  ```
  This shows that `persistentvolumeclaims` are scoped within a namespace.

---

### **Why Is This Important?**

Let’s say you have a namespace called `app1-ns`. If a PVC is created in `app1-ns` and binds to a PV, **only Pods in `app1-ns` can use that PVC**.

If a Pod in `app2-ns` tries to reference the same PVC, it will fail — because the PVC is invisible and inaccessible outside its namespace.

---

### **1-to-1 Binding Relationship Between PVC and PV**

- A **PVC can bind to only one PV**.
- Similarly, **a PV can be bound to only one PVC**.
- This is a **strict one-to-one relationship**, ensuring data integrity and predictable access control.
- Once a PV is bound, its `claimRef` field is populated, and it cannot be claimed by any other PVC unless explicitly released.

> **`claimRef`** is a field in a **PersistentVolume (PV)** that records which **PersistentVolumeClaim (PVC)** has successfully claimed it. It includes details like the PVC’s name and namespace. This field ensures that the PV is not mistakenly claimed by any other PVC, enforcing a **one-to-one binding** between the PV and its assigned PVC.

---

### **Additional Key Points**
- **PVCs request storage**; PVs **fulfill that request** if they match capacity, access mode, and storage class.
- Once a PVC is bound, **it remains bound** until:
  - The PVC is deleted.
  - The PV is manually reclaimed or deleted (depending on the reclaim policy).
- The reclaim policy (`Retain`, `Delete`, or deprecated `Recycle`) determines what happens to the PV after the PVC is deleted.

---

### **Example Scenario**

Let’s say:
- You create a PVC named `data-pvc` in the namespace `app1-ns`.
- It binds to a cluster-scoped PV.
- Only **Pods in `app1-ns`** can now reference this PVC.

If a Pod in `app2-ns` tries to mount this PVC, it will result in an error like:
```
persistentvolumeclaims "data-pvc" not found
```

Because from `app2-ns`'s perspective, that PVC does not exist.

---

### **Kubernetes Persistent Storage Flow (Manual Provisioning)**

![Alt text](/images/27c.png)

| **Step** | **Role**                | **Action**                                                                                          | **Details / Notes**                                                                 |
|----------|-------------------------|------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| 1        | **Developer**           | Requests 5Gi persistent storage for a Pod.                                                           | May request via a PVC or through communication with the Kubernetes Admin.          |
| 2        | **Kubernetes Admin**    | Coordinates with Storage Admin for backend volume.                                                  | Backend storage could be SAN/NAS exposed via iSCSI, NFS, etc.                      |
| 3        | **Storage Admin**       | Allocates a physical volume from a 500Ti storage pool.                                               | May involve LUN creation, NFS export, etc., based on the infrastructure.            |
| 4        | **Kubernetes Admin**    | Creates a **PersistentVolume (PV)** representing the physical volume in Kubernetes.                 | Specifies capacity, `accessModes`, `volumeMode`, `storageClassName`, etc.          |
| 5        | **Developer**           | Creates a **PersistentVolumeClaim (PVC)** requesting 5Gi with specific access and volume modes.     | PVC must match criteria defined in the PV.                                          |
| 6        | **Kubernetes**          | Binds PVC to a suitable PV if all parameters match.                                                 | Matching criteria include: storage class, access mode, volume mode, size, etc.     |
| 7        | **Pod**                 | References the PVC in its volume definition and mounts it in a container.                          | PVC acts as an abstraction; Pod doesn’t interact with the PV directly.             |

---

### **Important Notes**

- **PV** is a **cluster-scoped** resource.
- **PVC** is a **namespaced** resource.
- One **PV can be claimed by only one PVC** (1:1 relationship).
- The Pod must be in the **same namespace** as the PVC it is using.
- Communication with physical storage is handled by either:
  - **In-tree drivers** (legacy; e.g., `awsElasticBlockStore`, `azureDisk`)
  - **CSI drivers** (modern; e.g., `ebs.csi.aws.com`, `azurefile.csi.azure.com`)

> In many cases, developers are well-versed with Kubernetes and can handle the creation of **PersistentVolumeClaims (PVCs)** themselves. With the introduction of **StorageClasses**, the process of provisioning **PersistentVolumes (PVs)** has been **automated**—eliminating the need for Kubernetes administrators to manually coordinate with storage admins and pre-create PVs. When a PVC is created with a **StorageClass**, Kubernetes **dynamically provisions** the corresponding PV. We’ll explore StorageClasses in detail shortly.

---

#### **Access Modes in Kubernetes Persistent Volumes**

Persistent storage in Kubernetes supports various access modes that dictate how a volume can be mounted. Access modes essentially govern how the volume is mounted across **nodes**, which is critical in clustered environments like Kubernetes.
 

| **Access Mode**          | **Description**                                                                                          | **Example Use Case**                                            | **Type of Storage & Examples** |
|--------------------------|----------------------------------------------------------------------------------------------------------|----------------------------------------------------------------|--------------------------------|
| **ReadWriteOnce (RWO)**  | The volume can be mounted as read-write by a **single node**. Multiple Pods can access it **only if** they are on the same node. | Databases that require exclusive access but may run multiple replicas per node.  | **Block Storage** (e.g., Amazon EBS, GCP Persistent Disk, Azure Managed Disks) |
| **ReadOnlyMany (ROX)**   | The volume can be mounted as **read-only** by **multiple nodes** simultaneously.                        | Sharing static data like configuration files or read-only datasets across multiple nodes. | **File Storage** (e.g., NFS, Azure File Storage) |
| **ReadWriteMany (RWX)**  | The volume can be mounted as **read-write** by **multiple nodes** simultaneously.                       | Content management systems, shared data applications, or log aggregation. | **File Storage** (e.g., Amazon EFS, Azure File Storage, On-Prem NFS) |
| **ReadWriteOncePod (RWOP)** (Introduced in v1.29) | The volume can be mounted as read-write by **only one Pod across the entire cluster**.                 | Ensuring exclusive access to a volume for a single Pod, such as in tightly controlled workloads. | **Block Storage** (e.g., Amazon EBS with `ReadWriteOncePod` enforcement) |

---

### **Explanation of Storage Types**

#### **Block Storage**  
Block storage is ideal for databases and applications requiring **low-latency, high-performance storage**. It provides raw storage blocks that applications can format and manage as individual disks.  
- **Examples**: Amazon EBS, GCP Persistent Disk, Dell EMC Block Storage.  
- **Key Characteristic**: Block storage is generally **node-specific** and does not support simultaneous multi-node access.  
- **Access Modes**: Commonly used with `ReadWriteOnce` or `ReadWriteOncePod`, as these modes restrict access to a single node or Pod at a time.

*Analogy*: Block storage is like attaching a USB drive to a single computer—it provides fast, reliable storage but cannot be shared concurrently across multiple systems.

---

#### **File Storage**  
File storage is designed for **shared storage scenarios**, where multiple Pods or applications need simultaneous access to the same data. It is mounted as a shared filesystem, making it ideal for distributed workloads.  
- **Examples**: Amazon EFS, Azure File Storage, On-Prem NFS (Network File System).  
- **Key Characteristic**: File storage is purpose-built for **multi-node concurrent access**.  
- **Access Modes**: File storage often supports modes like `ReadOnlyMany` or `ReadWriteMany`, allowing multiple Pods—across different nodes—to read from and write to the same volume.

*Analogy*: File storage works like a network drive, where multiple systems can access, update, and share files simultaneously.

---

### **Key Differences: Block Storage vs. File Storage**
1. **Multi-Node Access**: Block storage is single-node focused, whereas file storage allows concurrent access across multiple nodes.  
2. **Access Modes**: `ReadWriteOnce` or `ReadWriteOncePod` are typical for block storage, while `ReadWriteMany` is common for file storage due to its multi-node capabilities.  
3. **Use Cases**:  
   - **Block Storage**: Databases, transactional systems, or workloads requiring exclusive and high-performance storage.  
   - **File Storage**: Shared workloads like web servers, content management systems, and applications requiring shared configurations or assets.

When evaluating storage options, it's important to align the access modes and storage type with the needs of the workload. For example, "Many" in an access mode (`ReadOnlyMany` or `ReadWriteMany`) usually signals that the underlying storage is file-based and optimized for shared use.

---


### **Reclaim Policies in Kubernetes**  

Reclaim policies define what happens to a **PersistentVolume (PV)** when its bound **PersistentVolumeClaim (PVC)** is deleted. The available policies are:  


#### **1. Delete (Common for Dynamically Provisioned Storage)**  
- When the PVC is deleted, the corresponding PV and its underlying **storage resource** (e.g., cloud disk, block storage) are **automatically deleted**.  
- This is useful in **cloud environments** where storage resources should be freed when no longer in use.  

**🔹 Example Use Case:**  
- **AWS EBS, GCP Persistent Disk, Azure Disk** – Storage dynamically provisioned via CSI drivers gets deleted along with the PV, preventing orphaned resources.  

---

#### **2. Retain (Manual Intervention Needed for Reuse)**  
- When the PVC is deleted, the PV remains in the cluster but moves to a **"Released"** state.  
- **The data is preserved**, and manual intervention is required to either:  
  - Delete and clean up the volume.  
  - Rebind it to another PVC by manually removing the claim reference (`claimRef`).  

**🔹 Example Use Case:**  
- **Auditing & Compliance:** Ensures data is retained for logs, backups, or forensic analysis.  
- **Manual Data Recovery:** Useful in scenarios where storage should not be automatically deleted after PVC removal.  

---

#### **3. Recycle (Deprecated in Kubernetes v1.20+)**  
- This policy would automatically **wipe the data** (using a basic `rm -rf` command) and make the PV available for new claims.  
- It was removed in favor of **dynamic provisioning** and more secure, customizable cleanup methods.  

**🔹 Why Deprecated?**  
- Lacked customization for **secure erasure** methods.  
- Didn't support advanced cleanup operations (e.g., snapshot-based restoration).  

---

### **Choosing the Right Reclaim Policy**  

| **Reclaim Policy** | **Behavior** | **Best Use Case** | **Common in** |
|-------------------|------------|-----------------|----------------|
| **Delete** | Deletes PV and storage resource when PVC is deleted. | Cloud-based dynamically provisioned storage. | AWS EBS, GCP PD, Azure Disk. |
| **Retain** | Keeps PV and storage, requiring manual cleanup. | Backup, auditing, manual data recovery. | On-prem storage, long-term retention workloads. |
| **Recycle (Deprecated)** | Cleans volume and makes PV available again. | (Not recommended) | Previously used in legacy systems. |


---

### **PVC and PV Binding Conditions**  

For a **PersistentVolumeClaim (PVC)** to bind with a **PersistentVolume (PV)** in Kubernetes, the following conditions must be met:  

- **Matching Storage Class**  
  - The `storageClassName` of the PVC and PV must match.  
  - If the PVC does not specify a storage class, it can bind to a PV **without a storage class**.  

- **Access Mode Compatibility**  
  - The access mode requested by the PVC (`ReadWriteOnce`, `ReadOnlyMany`, `ReadWriteMany`) **must be supported** by the PV.  

- **Sufficient Storage Capacity**  
  - The PV’s storage **must be equal to or greater than** the requested capacity in the PVC.  

- **Volume Binding Mode**  
  - If set to `Immediate`, the PV binds as soon as a matching PVC is found.  
  - If set to `WaitForFirstConsumer`, binding happens **only when a pod** using the PVC is scheduled.  

- **PV Must Be Available**  
  - The PV must be in the `Available` state (i.e., not already bound to another PVC).  
  - If the PV is already bound, it **cannot** be reused unless manually released.  

- **Matching Volume Mode**

    **Volume Modes** define how a Persistent Volume (PV) is presented to a Pod:
    1. **Block**: Provides raw, unformatted storage for the Pod. The application handles formatting and usage.  
    2. **Filesystem**: Presents a formatted volume, ready for file-level operations.  

    **Matching Modes**:  
    - A PVC requesting `volumeMode: Block` must match a PV with `volumeMode: Block`.  
    - A PVC requesting `volumeMode: Filesystem` must match a PV with `volumeMode: Filesystem`.

    **Use Case for `volumeMode: Block`**: This is typically used when an application, such as a database (e.g., PostgreSQL, MySQL), needs direct control over disk formatting, partitioning, or low-level I/O optimizations.

    This ensures compatibility between Pods and their storage. 

- **Claim Reference (Manual Binding Cases)**  
  - If the PV has a `claimRef` field, it can **only bind** to the specific PVC mentioned in that field.  

These conditions ensure a **seamless** and **reliable** binding process, providing persistent storage to Kubernetes workloads.  

---

### **Summary Table: PVC and PV Binding Conditions**  

| **Condition**              | **Requirement for Binding**                                             |
|----------------------------|-------------------------------------------------------------------------|
| **Storage Class Match**     | `storageClassName` of PVC and PV must match (or both can be empty).   |
| **Access Mode Compatibility** | PVC’s requested access mode must be supported by PV.                 |
| **Sufficient Capacity**     | PV’s storage must be **≥** PVC’s requested capacity.                 |
| **Volume Binding Mode**     | Either `Immediate` or `WaitForFirstConsumer`.                         |
| **Volume State**           | PV must be in `Available` state to bind.                             |
| **Matching Volume Mode**    | PVC and PV must have the same `volumeMode` (`Filesystem` or `Block`). |
| **Claim Reference**         | If PV has a `claimRef`, it can only bind to that specific PVC.        |

---

### **Example Table: PVC vs. PV Matching**  

| **Condition**       | **PVC Requirement** | **PV Must Have**        |
|--------------------|---------------------|-------------------------|
| **Storage Capacity** | `size: 10Gi`        | `size ≥ 10Gi`           |
| **Access Mode**      | `ReadWriteMany`     | `ReadWriteMany`         |
| **Storage Class**    | `fast-ssd`          | `fast-ssd`              |
| **Volume State**     | `Unbound`           | `Available`             |
| **Volume Mode**      | `Filesystem`        | `Filesystem`            |

---

### **Critical Note for KIND/Minikube Users**

If you're following along with this course, chances are you’ve installed **KIND (Kubernetes IN Docker)**. KIND comes with a **pre-configured default StorageClass** out of the box.  

If you're using **Minikube** instead, it's a good idea to check whether your cluster also includes a default StorageClass. You can verify this using the following command:

```bash
kubectl get storageclasses
```

Example output:
```
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  27d
```

#### **Why Modify the Default Storage Class?**

The default storage class (`standard`) interferes with our demo of Persistent Volumes (PVs) and Persistent Volume Claims (PVCs). For this reason, we will temporarily **delete it**. However, before deleting it, we’ll take a backup of the YAML configuration. This will allow us to recreate the storage class later when moving to the **Storage Classes** section.

---

#### **Steps to Back Up and Delete the Storage Class**

1. **Backup the Default Storage Class Configuration**:
   Use the following command to back up the configuration into a file named `sc.yaml` in your current working directory:
   ```bash
   kubectl get sc standard -o yaml > sc.yaml
   ```
   - This ensures we can recreate the `standard` storage class later as needed.

2. **Delete the Storage Class**:
   Now, delete the `standard` storage class to prevent interference with the PV/PVC demo:
   ```bash
   kubectl delete sc standard
   ```
   Example output:
   ```
   storageclass.storage.k8s.io "standard" deleted
   ```

By following these steps, we ensure that the default configuration doesn’t disrupt our hands-on exercises and we can restore it later when necessary.

---

### **Demo: Persistent Volumes and PVCs with Reclaim Policy**

#### **Step 1: Create a Persistent Volume (PV)**

Create a file (for example, `pv.yaml`) with the following content:

```yaml
apiVersion: v1                       # Kubernetes API version
kind: PersistentVolume              # Defines a PersistentVolume resource
metadata:
  name: example-pv                  # Unique name for the PV
spec:
  capacity:
    storage: 5Gi                    # Total storage provided (5 GiB)
  accessModes:
    - ReadWriteOnce                # Volume can be mounted as read-write by a single node at a time
  persistentVolumeReclaimPolicy: Retain  # Retain the volume and data even when the PVC is deleted
  hostPath:
    path: /mnt/data                # Uses a directory on the node (for demo purposes only)
```

Sure! Here's an improved and clearer version of your note:

> **Note:** When the `ReclaimPolicy` is set to `Retain`, the PersistentVolume (PV) and its data will **not be deleted** even if the associated PersistentVolumeClaim (PVC) is removed. This means the storage is **preserved for manual recovery or reassignment**, and must be manually handled by an administrator before it can be reused.


**Apply the PV:**

```bash
kubectl apply -f pv.yaml
```

**Verify the PV:**

```bash
kubectl get pv
kubectl describe pv example-pv
```

---

#### **Step 2: Create a Persistent Volume Claim (PVC)**

Create a file (for example, `pvc.yaml`) with the following content:

```yaml
apiVersion: v1                       # Kubernetes API version
kind: PersistentVolumeClaim         # Defines a PVC resource
metadata:
  name: example-pvc                 # Unique name for the PVC
spec:
  accessModes:
    - ReadWriteOnce                # Request volume to be mounted as read-write by a single node
  resources:
    requests:
      storage: 2Gi                 # Ask for at least 2Gi of storage (must be ≤ PV capacity)
```

> **Key Point:**  
> Since this PVC doesn’t explicitly specify a StorageClass, it will bind to a compatible PV if available. In this demo, the PV created above offers 5Gi, making it a suitable candidate for a 2Gi claim.

**Apply the PVC:**

```bash
kubectl apply -f pvc.yaml
```

**Verify the PVC status:**

```bash
kubectl get pvc
kubectl describe pvc example-pvc
```

---

#### **Step 3: Create a Pod That Uses the PVC**

Create a file (for example, `pod.yaml`) with the following content:

```yaml
apiVersion: v1                       # Kubernetes API version
kind: Pod                           # Defines a Pod resource
metadata:
  name: example-pod                 # Unique name for the Pod
spec:
  containers:
    - name: nginx-container         # Name of the container
      image: nginx                  # Container image to use
      volumeMounts:
        - mountPath: /usr/share/nginx/html  # Directory inside the container where the volume will be mounted
          name: persistent-storage  # Logical name for the volume mount
  volumes:
    - name: persistent-storage      # Volume's name referenced above
      persistentVolumeClaim:
        claimName: example-pvc      # Bind this volume to the previously created PVC
```

> **Important:**  
> When this Pod is created, Kubernetes will bind the PVC to the appropriate PV (if not already bound) and mount the volume. At this point, the PVC status should change from "Pending" to "Bound".

**Apply the Pod:**

```bash
kubectl apply -f pod.yaml
```

**Verify the Pod and its Volume Attachment:**

```bash
kubectl describe pod example-pod
```

---

#### **Final Verification**

After creating these resources, use the following commands to check that everything is in order:

- **Persistent Volumes:**
  ```bash
  kubectl get pv
  kubectl describe pv example-pv
  ```
- **Persistent Volume Claims:**
  ```bash
  kubectl get pvc
  kubectl describe pvc example-pvc
  ```
- **Pod Details:**
  ```bash
  kubectl describe pod example-pod
  ```

By following these steps, you’ll see that the PVC is bound to the PV and the Pod successfully mounts the storage. This demo illustrates how the `Retain` reclaim policy preserves data on the PV and how the dynamic binding between PVCs and PVs works within Kubernetes.

---

### **Storage Classes & Dynamic Provisioning**
---
#### **What is a Storage Class?**

A **Storage Class** in Kubernetes is a way to define different storage configurations, enabling dynamic provisioning of Persistent Volumes (PVs). It eliminates the need to manually pre-create PVs and provides flexibility for managing storage across diverse workloads. 

- **Purpose**: Storage Classes define storage backends and their parameters, such as disk types, reclaim policies, and binding modes.  
- **Dynamic Provisioning**: When a Persistent Volume Claim (PVC) is created, Kubernetes uses the referenced Storage Class to automatically provision a corresponding PV.  
- **Flexibility**: Multiple Storage Classes can coexist in a Kubernetes cluster, allowing administrators to tailor storage types for varying application needs (e.g., high-performance SSDs, low-cost storage, etc.).

---

#### **Why Is a Storage Class Required?**

1. Simplifies the storage lifecycle by automating PV creation using dynamic provisioning.  
2. Offers flexibility to define and manage multiple storage tiers.  
3. Optimizes storage resource allocation, especially in environments spanning multiple Availability Zones (AZs).

> StorageClass takes over the role of provisioning PVs dynamically, replacing many of the static configurations you used to define in PVs manually. But not everything from PV moves into the StorageClass—some things like **access modes, size, volumeMode** still come from PVC.

---

### **Example Storage Classes**

Below are two examples of AWS EBS Storage Classes, demonstrating how multiple classes can coexist in the same cluster:

---

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc-gp3  # Name of the StorageClass for AWS EBS gp3 volumes.
provisioner: ebs.csi.aws.com  # Specifies the CSI driver for AWS EBS.
parameters:
  type: gp3  # Defines the volume type as gp3 (general purpose SSD with configurable performance).
reclaimPolicy: Delete  # Deletes the provisioned volume when the PVC is deleted.
volumeBindingMode: WaitForFirstConsumer  # Delays volume creation until the Pod is scheduled.
```

---

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc-io1  # Name of the StorageClass for AWS EBS io1 volumes.
provisioner: ebs.csi.aws.com  # Specifies the CSI driver for AWS EBS.
parameters:
  type: io1  # Defines the volume type as io1 (high-performance SSD).
reclaimPolicy: Delete  # Deletes the provisioned volume when the PVC is deleted.
volumeBindingMode: WaitForFirstConsumer  # Ensures the volume is created in the same AZ as the Pod.
```

---

#### **Key Points**

1. **Reclaim Policy**:
   - The `Delete` reclaim policy ensures that dynamically provisioned volumes are automatically cleaned up when their corresponding PVCs are deleted.  
   - This prevents orphaned resources and is the most common choice for dynamically provisioned storage.

2. **WaitForFirstConsumer**:

![Alt text](/images/27d.png)
   - In a Kubernetes cluster spanning multiple Availability Zones (AZs), **EBS volumes and EC2 instances are AZ-specific resources**.  
   - If a volume is immediately provisioned in one AZ when a PVC is created, and the Pod using the PVC is scheduled in another AZ, the volume cannot be mounted.  
   - The `WaitForFirstConsumer` mode ensures that the volume is created **only after the Pod is scheduled**, ensuring both the Pod and the volume are in the same AZ.  
   - This prevents inefficiencies and reduces unnecessary costs for resources provisioned in the wrong AZ.

---

### **Dynamic Provisioning in Action**

Let’s see how the `ebs-sc-gp3` Storage Class is used with a PVC, a dynamically provisioned PV, and a Pod.

#### Persistent Volume Claim (PVC)
The PVC requests dynamic provisioning by referencing the `ebs-sc-gp3` Storage Class.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-csi-pvc  # Name of the PVC to be used by Pods.
spec:
  accessModes:
    - ReadWriteOnce  # The volume can be mounted as read-write by a single node.
  resources:
    requests:
      storage: 10Gi  # Minimum storage capacity requested.
  storageClassName: ebs-sc-gp3  # References the gp3 StorageClass for dynamic provisioning.
```

---

#### Persistent Volume (PV)
This is an example of a PV **dynamically** created by the CSI driver when the above PVC is applied.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ebs-csi-pv  # Name of the dynamically provisioned Persistent Volume.
spec:
  capacity:
    storage: 10Gi  # Defines the storage capacity for the volume.
  volumeMode: Filesystem  # Specifies the volume is presented as a filesystem (default mode).
  accessModes:
    - ReadWriteOnce  # Restricts volume to a single node for read-write operations.
  persistentVolumeReclaimPolicy: Delete  # Automatically deletes the volume when the PVC is deleted.
  storageClassName: ebs-sc-gp3  # Matches the StorageClass that provisioned this PV.
  csi:
    driver: ebs.csi.aws.com  # The AWS EBS CSI driver responsible for provisioning this volume.
    volumeHandle: vol-0abcd1234efgh5678  # Identifies the volume in the AWS backend.
    fsType: ext4  # The filesystem type for the volume.
```

---

#### Pod Using PVC
The Pod dynamically mounts the volume provisioned by the PVC.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ebs-csi-pod  # Name of the Pod.
spec:
  containers:
    - name: app-container  # Name of the container in the Pod.
      image: nginx  # The container image to run.
      volumeMounts:
        - mountPath: /usr/share/nginx/html  # Mounts the volume at this path inside the container.
          name: ebs-storage  # References the volume defined in the Pod spec.
  volumes:
    - name: ebs-storage  # Volume name referenced in the container's volumeMounts.
      persistentVolumeClaim:
        claimName: ebs-csi-pvc  # Links the volume to the PVC created earlier.
```

---

#### **Key Takeaways**

- Storage Classes simplify storage management in Kubernetes, allowing dynamic provisioning of Persistent Volumes based on application needs.  
- The `reclaimPolicy: Delete` ensures proper cleanup of volumes once they are no longer needed.  
- The `WaitForFirstConsumer` binding mode optimizes placement and ensures resources like EBS volumes and Pods are aligned in multi-AZ environments.  
- By combining Storage Classes, PVCs, and dynamic provisioning, Kubernetes provides a powerful and flexible storage solution for managing workloads efficiently. 

---

### **Demo: Storage Class**

#### **Step 1: Reapply the Storage Class**
Before proceeding with the demo, we need to restore the `StorageClass` configuration that we backed up (`sc.yaml`). Run the following command to reapply it:

```bash
kubectl apply -f sc.yaml
```

This re-establishes the default `standard` StorageClass in your KIND cluster.

---

#### **Step 2: Create the PersistentVolumeClaim (PVC)**

Below is the YAML to create a PVC. It requests storage but does not explicitly reference any StorageClass:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc  # Name of the PersistentVolumeClaim
spec:
  accessModes:
    - ReadWriteOnce  # The volume can be mounted as read-write by a single node.
  resources:
    requests:
      storage: 2Gi  # Requests a minimum of 2Gi storage capacity.
```

**Key Explanation**:
- Even though we didn’t specify a StorageClass, Kubernetes defaults to using the `standard` StorageClass (if one is configured as the default).  
- The **status of the PVC** will remain as **"Pending"** initially since no Persistent Volume (PV) is created at this point.

To understand why the PVC is pending, describe the StorageClass with:
```bash
kubectl describe sc standard
```

You’ll see that the `standard` StorageClass is configured as default:
```yaml
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: rancher.io/local-path
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

---

#### **Step 3: Understand VolumeBindingMode**
The `WaitForFirstConsumer` mode plays a critical role:
- It delays PV creation **until a Pod is scheduled**, ensuring cost optimization and proper resource placement.  
- For example, in multi-AZ environments like AWS, if the PVC triggers volume creation in **AZ-1** but the Pod is scheduled in **AZ-2**, the volume won’t be accessible. `WaitForFirstConsumer` avoids this by creating the volume **only after a Pod is scheduled**, ensuring both the Pod and volume are in the same AZ.

---

#### **Step 4: Create a Pod Using the PVC**

Below is the YAML to create a Pod that uses the PVC:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod  # Name of the Pod
spec:
  containers:
    - name: nginx-container  # Container name
      image: nginx  # The container image to use
      volumeMounts:
        - mountPath: /usr/share/nginx/html  # Mounts the volume to this path in the container
          name: persistent-storage  # References the volume defined in the Pod
  volumes:
    - name: persistent-storage  # Name of the volume
      persistentVolumeClaim:
        claimName: example-pvc  # Links the PVC to the Pod volume
```

**Key Explanation**:
- Once the Pod is created, Kubernetes finds the PVC (`example-pvc`) and provisions a PV using the default `standard` StorageClass.  
- The PVC status changes to **Bound**, and a new PV is created and attached to the Pod.

---

#### **Step 5: Verify the Status**

Run the following commands to check the status of PVs and PVCs:

1. **Check PVs**:
   ```bash
   kubectl get pv
   ```
   Example output:
   ```
   NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   AGE
   pvc-24d1f4ee-d3f8-40eb-8120-21f232087a19   2Gi        RWO            Delete           Bound    default/example-pvc   standard       6m
   ```

2. **Check PVCs**:
   ```bash
   kubectl get pvc
   ```
   Example output:
   ```
   NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
   example-pvc   Bound    pvc-24d1f4ee-d3f8-40eb-8120-21f232087a19   2Gi        RWO            standard       6m
   ```

---

### **Key Takeaways**

1. **Default StorageClass**:
   - If no StorageClass is specified in the PVC, Kubernetes uses the default StorageClass (`standard`, in this case).  
   - The `is-default-class` annotation ensures it acts as the default.

2. **VolumeBindingMode (`WaitForFirstConsumer`)**:
   - Prevents PV creation until a Pod is scheduled, optimizing resource placement and cost in multi-AZ environments.

3. **Reclaim Policy (`Delete`)**:
   - Automatically deletes PVs once their associated PVCs are deleted, preventing storage clutter.

By following these steps, you can understand how dynamic provisioning works in Kubernetes with StorageClasses, PVCs, and Pods.

---

## ConfigMaps

### **Why Do We Need ConfigMaps?**

In real-world Kubernetes deployments:

- Maintaining a large number of hardcoded environment variables in Pod specs can become unmanageable.
- You may want to update configuration without rebuilding your container image.
- You might need to inject configuration data via environment variables, CLI arguments, or mounted files.

**ConfigMaps** allow you to **decouple configuration from your container images**, making your applications more portable and easier to manage.

---

### **What is a ConfigMap?**

A **ConfigMap** is a Kubernetes API object used to store **non-confidential key-value configuration data**. 

Pods can consume ConfigMaps in three primary ways:

1. As **environment variables**
2. As **command-line arguments** (Less Common)
3. As **configuration files via mounted volumes**

> **Important:** ConfigMaps do not provide encoding. For sensitive information, use a **Secret** instead.



**What are Environment Variables?**

Environment variables are key-value pairs used by the operating system and applications to store configuration settings. They help control how processes behave without modifying the underlying code.

**Example on a Linux system:**

```bash
export APP_NAME=frontend
export ENVIRONMENT=production
```

You can access these values using the `echo` command:

```bash
echo $APP_NAME     # Output: frontend
echo $ENVIRONMENT  # Output: production
```

These variables are commonly used in shell scripts or passed to applications during execution to configure them dynamically.

---

### **Without ConfigMap: Hardcoded Environment Variables**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend-container
          image: nginx
          env:
            - name: APP
              value: frontend
            - name: ENVIRONMENT
              value: production
```

While this works, hardcoding environment variables is not scalable or reusable across environments.

---

## **Demo 1: Using ConfigMap as Environment Variables**

For this demo, we’ll build on the `frontend-deploy` configuration introduced in **Day 12** of this course. If you'd like a deeper understanding of **Kubernetes Services**, feel free to explore the following resources:

- [Day 12 GitHub Notes](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2012)

- [Day 12 YouTube Video](https://www.youtube.com/watch?v=92NB8oQBtnc)

### Step 1: Create ConfigMap

```yaml
apiVersion: v1  # Defines the API version for the Kubernetes object.
kind: ConfigMap  # Declares this resource as a ConfigMap.
metadata:
  name: frontend-cm  # The name used to reference this ConfigMap in pods or other resources.
data:
  APP: "frontend"  # Key-value pair representing configuration data (e.g., application name).
  ENVIRONMENT: "production"  # Key-value pair used to define the deployment environment.

```

Apply:

```bash
kubectl apply -f frontend-cm.yaml
```

> **Note:** Always apply the **ConfigMap before the Deployment**, so that the pods can reference and consume the configuration data during startup.

---

### Step 2: Reference ConfigMap in Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deploy  # Name of the Deployment resource
spec:
  replicas: 3  # Number of pod replicas to maintain
  selector:
    matchLabels:
      app: frontend  # Selector to match pods managed by this Deployment
  template:
    metadata:
      labels:
        app: frontend  # Label assigned to pods created by this Deployment
    spec:
      containers:
      - name: frontend-container  # Name of the container within each pod
        image: nginx  # Using the official Nginx image
        env:
        - name: APP  # Environment variable name inside the container
          valueFrom:
            configMapKeyRef:
              name: frontend-cm  # Name of the ConfigMap from which the value is pulled
              key: APP  # Key in the ConfigMap whose value is used for this environment variable
        - name: ENVIRONMENT  # Another environment variable inside the container
          valueFrom:
            configMapKeyRef:
              name: frontend-cm
              key: ENVIRONMENT

        # NOTE:
        # The value of key "APP" will be picked up from the key "APP" in the ConfigMap named "frontend-cm".
        # The 'name' field here (APP) is the environment variable name that will be visible inside the container.
        # It does NOT need to match the 'key' from the ConfigMap.
        # For example:
        # - You could write: name: SERVICE and key: APP
        # - Inside the container, the environment variable would be called SERVICE and hold the value of APP from the ConfigMap.
```

Apply:

```bash
kubectl apply -f frontend-deploy.yaml
```

---

### Verification

Pick a Pod from the Deployment:

```bash
kubectl get pods -l app=frontend
```

Then exec into one of them:

```bash
kubectl exec -it <pod-name> -- printenv | grep -E 'APP|ENVIRONMENT'
```

Expected Output:

```
APP=frontend
ENVIRONMENT=production
```

---

**NOTE:** I forgot to cover this in the demo, but you can also **create a ConfigMap imperatively** using the `kubectl create configmap` command.  
For example:

```bash
kubectl create configmap <name-of-configmap> --from-literal=key1=value1 --from-literal=key2=value2
```

A real-world example would be:

```bash
kubectl create configmap frontend-cm --from-literal=APP=frontend --from-literal=ENVIRONMENT=production
```

> This method is quick and useful for creating simple ConfigMaps **imperatively** from the command line, without needing to write and apply a YAML manifest.

**When to Use Imperative vs Declarative Approach**

- **Imperative Commands (`kubectl create configmap`)** are best when you:
  - Need to quickly create a ConfigMap for testing or small demos.
  - Are experimenting or working interactively.
  - Don't need to track the resource in version control (like Git).

- **Declarative Approach (YAML manifests + `kubectl apply -f`)** is preferred when you:
  - Are working in production environments.
  - Want your ConfigMaps (and all Kubernetes resources) to be version-controlled.
  - Need better team collaboration, auditing, and repeatable deployments.

> **Rule of Thumb:**  
> Use **imperative** for quick, temporary tasks.  
> Use **declarative** for production-grade, repeatable, and auditable setups.

---

## **Demo 2: Using ConfigMap as a Configuration File (Volume Mount)**

For this demo, we’ll build on the `frontend-deploy` configuration introduced in **Day 12** of this course. If you'd like a deeper understanding of **Kubernetes Services**, feel free to explore the following resources:

- [Day 12 GitHub Notes](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2012)

- [Day 12 YouTube Video](https://www.youtube.com/watch?v=92NB8oQBtnc)

### Step 1: Modify ConfigMap to Include HTML File

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-cm  # Name of the ConfigMap, used for reference in Pods or Deployments.
data:
  APP: "frontend"  # Key-value pair that can be used as an environment variable in a Pod.
  ENVIRONMENT: "production"  # Another environment variable for defining the environment.
  index.html: |  # When mounted as a volume, this key creates a file named 'index.html' with the following contents.
    <!DOCTYPE html>
    <html>
    <head><title>Welcome</title></head>
    <body>
      <h1>This page is served by nginx. Welcome to Cloud With VarJosh!!</h1>
    </body>
    </html>
```
The `|` tells YAML:

> “Treat the following lines as a **literal block of text**. Preserve all line breaks and indentation exactly as they appear.”

In YAML, the `|` symbol is called a **block scalar indicator** — specifically, it's used for **literal block style**.
This is especially useful when you're writing multi-line values, such as **HTML**, **scripts**, or **configuration files** inside a YAML field.

Apply:

```bash
kubectl apply -f frontend-cm.yaml
```

---

### Step 2: Mount ConfigMap Volume in Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deploy  # Name of the Deployment
spec:
  replicas: 3  # Number of pod replicas
  selector:
    matchLabels:
      app: frontend  # Label selector to identify Pods managed by this Deployment
  template:
    metadata:
      labels:
        app: frontend  # Label to match with the selector above
    spec:
      containers:
      - name: frontend-container  # Name of the container
        image: nginx  # NGINX image to serve web content
        env:
        - name: APP
          valueFrom:
            configMapKeyRef:
              name: frontend-cm  # Name of the ConfigMap to pull the key from
              key: APP  # Key in the ConfigMap whose value will become the value of the env variable APP
        - name: ENVIRONMENT
          valueFrom:
            configMapKeyRef:
              name: frontend-cm
              key: ENVIRONMENT  # Key in the ConfigMap whose value becomes the value of ENVIRONMENT variable
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html  # Mount point inside the container (default HTML dir for NGINX)
          readOnly: true  # Make the volume read-only
      volumes:
      - name: html-volume
        configMap:
          name: frontend-cm  # The ConfigMap being mounted as a volume
          items:
          - key: index.html  # The key in the ConfigMap to be used as a file
            path: index.html  # The filename that will be created inside the mount path
```

Understand that **ConfigMaps are mounted as directories** in Kubernetes. If you try to mount a ConfigMap at a path that is already a file (like `/usr/share/nginx/html/index.html`), the container will **fail to start**, because Kubernetes **cannot mount a directory over an existing file**.

That’s why we only use `/usr/share/nginx/html` as the `mountPath`, which is a **directory**, not a file.

If you **omit the `items:` section**, like this:

```yaml
volumes:
  - name: html-volume
    configMap:
      name: frontend-cm
```

Then **all the keys** in the ConfigMap (`APP`, `ENVIRONMENT`, `index.html`) will be mounted as **individual files** inside `/usr/share/nginx/html`.

So inside the container, you’d get:
```
/usr/share/nginx/html/APP
/usr/share/nginx/html/ENVIRONMENT
/usr/share/nginx/html/index.html
```

If you **only want `index.html`** to be mounted (and not `APP`, `ENVIRONMENT`), you should use the `items:` section like this:

```yaml
volumes:
  - name: html-volume
    configMap:
      name: frontend-cm
      items:
        - key: index.html
          path: index.html
```
---

### What if you *do* want to mount the ConfigMap **as a single file**?

You cannot mount a directory on top of an existing file — but you **can mount a specific key** as a file using `subPath`. Here’s how:

```yaml
volumeMounts:
  - name: html-volume                    # References the volume named 'html-volume' defined in the 'volumes' section
    mountPath: /usr/share/nginx/html/index.html  # The exact file path inside the container where the file should be mounted
    subPath: index.html                  # Mount only the 'index.html' key from the ConfigMap as a file (not the whole directory)

```

This tells Kubernetes:  
> “Mount only the file named `index.html` from the ConfigMap into the container at `/usr/share/nginx/html/index.html`.”

This works even if `index.html` already exists in the NGINX image because you're **not replacing a file with a directory**, but a file with a file.

---

**Advantages of Using `subPath`**

- Allows **fine-grained control** over what file is mounted and where.
- Helps you **inject configuration** (like `index.html`) **without overriding entire directories**.
- Enables **mixing static content** from image and dynamic content from ConfigMap.


**Disadvantages of `subPath`**

- The mounted file is **not updated automatically** if the ConfigMap changes (unless the pod is restarted).
- Slightly more complex syntax.
- If the mount path already exists and is a file, mounting may fail without proper use of `subPath`.

---

So, in summary:

- Use `mountPath: /usr/share/nginx/html` to mount a **directory**.
- Use `subPath` to mount a **single file** precisely over a specific path.
- Use `items:` in the volume to **control which keys** from the ConfigMap are mounted.



Apply:

```bash
kubectl apply -f frontend-deploy.yaml
```

---

## **Step 3: Expose the Deployment using a NodePort Service**

We’ll now expose our NGINX-based frontend deployment using a **NodePort** service. This allows access to the application from outside the cluster by targeting any worker node’s IP and the assigned NodePort.

### NodePort Service Manifest

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc  # Name of the service
spec:
  type: NodePort       # Exposes the service on a static port on each node
  selector:
    app: frontend      # Targets pods with this label
  ports:
    - protocol: TCP
      port: 80         # Port exposed inside the cluster (ClusterIP)
      targetPort: 80   # Port the container is listening on
      nodePort: 31000  # External port accessible on the node (must be between 30000–32767)
```

Apply the service:

```bash
kubectl apply -f frontend-svc.yaml
```
---

## **Verification**

#### 1. Get the Pod name

```bash
kubectl get pods -l app=frontend
```

#### 2. Verify the `index.html` content inside the container

```bash
kubectl exec -it <pod-name> -- cat /usr/share/nginx/html/index.html
```

**Expected Output:**

```html
<!DOCTYPE html>
<html>
<head><title>Welcome</title></head>
<body>
  <h1>This page is served by nginx. Welcome to Cloud With VarJosh!!</h1>
</body>
</html>
```

---

#### 3. Access the Application via NodePort

You can access the application using:

```bash
http://<Node-IP>:31000
```

Or with `curl`:

```bash
curl http://<Node-IP>:31000
```

> Replace `<Node-IP>` with the IP address of any node in your cluster (typically a worker node).

---

#### 4. **If You're Using KIND (Kubernetes in Docker)**

If you've been following this course using a **KIND cluster**, the worker nodes are running inside a Docker container. In this case, KIND maps NodePorts to your host (Mac/Windows/Linux) via localhost.

**For more details, refer to the Day 12 GitHub notes:**  
[Setting up a KIND Cluster with NodePort Service](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2012#setting-up-a-kind-cluster-with-nodeport-service)

So you can directly access the app using:

```bash
http://localhost:31000
```

Or:

```bash
curl http://localhost:31000
```

This will display the content served by NGINX from the ConfigMap-mounted `index.html`.

--- 


### **Best Practices for ConfigMap**

- Avoid storing sensitive data in ConfigMaps.
- Mount only the necessary keys using `items` to limit volume content.
- Use version control tools (e.g., Kustomize, Helm) to manage changes.
- Combine ConfigMaps with `subPath` if you want to mount specific files.

---

## Kubernetes Secrets
### **Why Do We Need Secrets in Kubernetes?**

In production environments, applications often require access to sensitive data such as:

- **Database credentials:** Used by applications to authenticate securely with backend databases.
- **API tokens:** Serve as secure keys to authorize and access APIs or third-party services.
- **SSH private keys:** Enable secure, encrypted access to remote systems over SSH.
- **TLS certificates:** Provide encryption and identity verification for secure network communication (e.g., HTTPS).

Storing these directly in your container image or Kubernetes manifests as plain text is insecure. **Kubernetes Secrets** provide a way to manage this data securely.

---

### **What is a Kubernetes Secret?**

A **Secret** is a Kubernetes object used to store and manage sensitive information. Secrets are base64-encoded and can be made more secure by:

- Enabling encryption at rest
- Limiting RBAC access to Secrets
- Using external secret managers like HashiCorp Vault, AWS Secrets Manager, or Sealed Secrets

Accessible to Pods via:
  1. **Environment variables**
  2. **Mounted volumes (as files)**
  3. **Command-line arguments** (less common)

> Note: By default, Kubernetes stores secrets unencrypted in `etcd`. It is recommended to enable encryption at rest for better security.

---

### **Important Distinction: Encoding vs. Encryption**

It's crucial to understand that **Kubernetes Secrets use base64 *encoding*, not encryption**.  
This means the data is **obfuscated but not secured**. Anyone who gains access to the Secret object can **easily decode** it.

**Why Use Encoding (e.g., base64)?**

Encoding is useful when you want to **hide the data from casual observation**, such as:

- Preventing someone looking over your shoulder from instantly seeing a password.
- Making binary data safe to transmit in systems that expect text.

However, **encoding is not encryption**. It's **not secure** by itself.  
> Anyone who has access to your encoded data can easily decode it.

For example, base64 is **reversible** using a simple decoding command.  
If you need to **protect sensitive data**, you should use **encryption** or a Kubernetes **Secret**, which at least provides better handling and access controls.

We'll see how to **encode** and **decode** in the demo section.

---

### **Encoding vs. Encryption**

| Feature         | Encoding                        | Encryption                           |
|----------------|----------------------------------|--------------------------------------|
| **Purpose**     | Data formatting for safe transport | Data protection and confidentiality |
| **Reversible**  | Yes (easily reversible)          | Yes (only with the correct key)      |
| **Security**    | Not secure                       | Secure                                |
| **Use Case**    | Data transmission/storage compatibility | Protect sensitive data (passwords, tokens) |
| **Example**     | Base64, URL encoding             | AES, RSA, TLS                        |
| **Tool Needed to Decode** | None (any base64 tool)         | Requires decryption key              |

---

> **Note:** If you need to store sensitive data securely, consider enabling **encryption at rest** for Secrets in Kubernetes and restrict access using RBAC.

---

### **Demo 1: Injecting Secrets into a Pod**

We’ll enhance the existing `frontend-deploy` and `frontend-svc` resources by securely injecting database credentials using a Kubernetes Secret.

For this demo, we’ll build on the `frontend-deploy` configuration introduced in **Day 12** of this course. If you'd like a deeper understanding of **Kubernetes Services**, feel free to explore the following resources:

- [Day 12 GitHub Notes](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2012)  
- [Day 12 YouTube Video](https://www.youtube.com/watch?v=92NB8oQBtnc)

These resources provide clear examples and explanations of how Kubernetes Services work in real-world deployments.

---

### **Step 1: Create the Secret**

Create a Secret manifest `frontend-secret.yaml`:

```yaml
apiVersion: v1
kind: Secret  # Declares the resource type as a Secret.
metadata:
  name: frontend-secret  # Unique name for this Secret object.
type: Opaque  # 'Opaque' means this Secret contains user-defined (arbitrary) key-value pairs.

data:
  DB_USER: ZnJvbnRlbmR1c2Vy  # Base64-encoded value of 'frontenduser'
  DB_PASSWORD: ZnJvbnRlbmRwYXNz  # Base64-encoded value of 'frontendpass'

  # NOTE:
  # Values under the 'data:' section must be base64-encoded.
  # These can be referenced by Pods to inject as environment variables or mounted as files.
```

To generate the base64-encoded values:

```bash
echo -n 'frontenduser' | base64      # ZnJvbnRlbmR1c2Vy
echo -n 'frontendpass' | base64      # ZnJvbnRlbmRwYXNz
```

To decode the values on a Linux system:

```bash
echo 'ZnJvbnRlbmR1c2Vy' | base64 --decode   # frontenduser
echo 'ZnJvbnRlbmRwYXNz' | base64 --decode   # frontendpass
```

> **Note:** The `-n` flag with `echo` ensures no trailing newline is added before encoding.

Apply the Secret:

```bash
kubectl apply -f frontend-secret.yaml
```

---

### **Step 2: Update the Deployment to Use the Secret**

Here’s the updated deployment manifest (`frontend-deploy.yaml`) using the `frontend-secret` Secret as environment variables:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deploy  # Name of the Deployment
spec:
  replicas: 1  # Run a single replica of the frontend Pod
  selector:
    matchLabels:
      app: frontend  # This selector ensures the Deployment manages only Pods with this label
  template:
    metadata:
      labels:
        app: frontend  # Label added to the Pod; matches the selector above
    spec:
      containers:
        - name: frontend-container  # Name of the container running inside the Pod
          image: nginx  # Using official NGINX image
          env:
            - name: DB_USER  # Environment variable DB_USER inside the container
              valueFrom:
                secretKeyRef:
                  name: frontend-secret  # Name of the Secret resource
                  key: DB_USER  # Key in the Secret to pull the value from
            - name: DB_PASSWORD  # Environment variable DB_PASSWORD inside the container
              valueFrom:
                secretKeyRef:
                  name: frontend-secret  # Same Secret object
                  key: DB_PASSWORD  # Another key from the Secret object

  # The environment variables DB_USER and DB_PASSWORD will be injected into the container at runtime.
  # Kubernetes will automatically base64-decode the values from the Secret and inject them as plain-text.
  # That means inside the container, these environment variables will appear as plain-text,
  # even though they are stored in base64-encoded format in the Kubernetes Secret object.

```

Apply the deployment:

```bash
kubectl apply -f frontend-deploy.yaml
```

---

### **Step 3: Verify Secret Injection**

1. List the running Pods:

```bash
kubectl get pods -l app=frontend
```

2. Exec into one of the Pods:

```bash
kubectl exec -it <pod-name> -- /bin/sh
```

3. Print the environment variables:

```bash
echo $DB_USER       # Expected output: frontenduser
echo $DB_PASSWORD   # Expected output: frontendpass
```

---

### **Alternative: Mount Secret as Files**

If you prefer to mount secrets as files (instead of environment variables), here’s how:

#### Update container spec:

Certainly! Here's your `volumeMounts` section with clear and concise inline comments:

```yaml
        volumeMounts:
          - name: secret-volume               # Refers to the volume named 'secret-volume' defined in the 'volumes' section
            mountPath: /etc/secrets           # Directory inside the container where the secret files will be mounted
            readOnly: true                    # Ensures the mounted volume is read-only to prevent modifications from within the container
```
**Additional Notes:**

- Each key in the Secret will be mounted as a **separate file** under `/etc/secrets`.
- For example, if your Secret has `DB_USER` and `DB_PASSWORD`, you’ll find:
  ```bash
  /etc/secrets/DB_USER
  /etc/secrets/DB_PASSWORD
  ```
- These files will contain the **decoded plain-text values** from the Secret.


#### Add volume to spec:

Certainly! Here's the corresponding `volumes:` section with detailed inline comments to pair with the `volumeMounts` section above:

```yaml
      volumes:
        - name: secret-volume                # The volume name referenced in volumeMounts
          secret:
            secretName: frontend-secret     # The name of the Secret object from which to pull data

            # Optional: you could use 'items' here to mount only specific keys
            # For example:
            # items:
            #   - key: DB_USER
            #     path: db-user.txt
            #   - key: DB_PASSWORD
            #     path: db-password.txt

            # Without 'items', all keys from the Secret will be mounted as individual files
            # in the mountPath directory (e.g., /etc/secrets/DB_USER, /etc/secrets/DB_PASSWORD)
```

---

### **Best Practices for Using Kubernetes Secrets**

- Avoid storing secrets in Git repositories.
- Use `kubectl create secret` or Helm to avoid manually encoding data.
- Enable encryption at rest in `etcd`.
- Use Role-Based Access Control (RBAC) to restrict access to secrets.
- Use external secret managers for enhanced security and auditability.
- Consider using `subPath` when mounting specific keys from a Secret as individual files.

---

### **Understanding Dynamic Updates with ConfigMaps and Secrets**

- Environment variables defined via `env.valueFrom.configMapKeyRef` or `secretKeyRef` are **evaluated only once** when the pod starts.
- Updating the underlying ConfigMap or Secret **does not affect** the values already injected as environment variables in a running container.
- ConfigMaps or Secrets mounted as **volumes** (without `subPath`) **do reflect updates dynamically** inside the container.
- Kubernetes handles dynamic updates using **symlinks** to new file versions, but the application must **re-read the files** to detect changes.
- When mounting individual keys using `subPath`, the file is **copied**, not symlinked, so updates to the ConfigMap or Secret **will not propagate**.
- To enable live updates without restarting pods, prefer **volume mounts without `subPath`** and ensure the application supports **hot reloading** or use a **config-reloader**.

---

### Conclusion

By the end of this MASTERCLASS, you will have a strong practical and conceptual understanding of:

* How storage is managed in Docker using image layers, storage drivers, and volumes
* The differences between ephemeral and persistent storage in Kubernetes
* How to use volumes like `emptyDir`, `hostPath`, and CSI-backed Persistent Volumes
* How ConfigMaps and Secrets interact with volume mounts and environment variables
* The inner workings of Kubernetes' plugin-based architecture via CRI, CNI, and CSI
* Best practices around PVs, PVCs, StorageClasses, access modes, reclaim policies, and binding conditions

Whether you're deploying applications in development clusters or managing production-grade workloads, this foundational understanding of Kubernetes storage equips you to design scalable, portable, and resilient systems.

---

### References

Below are the **official references**, presented in the order concepts were covered:

1. Docker Storage Overview
   [https://docs.docker.com/storage/](https://docs.docker.com/storage/)

2. Docker Volume Types
   [https://docs.docker.com/storage/volumes/](https://docs.docker.com/storage/volumes/)

3. Docker Volume Drivers
   [https://docs.docker.com/engine/extend/legacy\_plugins/](https://docs.docker.com/engine/extend/legacy_plugins/)

4. Kubernetes Volumes
   [https://kubernetes.io/docs/concepts/storage/volumes/](https://kubernetes.io/docs/concepts/storage/volumes/)

5. `emptyDir` Volume
   [https://kubernetes.io/docs/concepts/storage/volumes/#emptydir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)

6. `hostPath` Volume
   [https://kubernetes.io/docs/concepts/storage/volumes/#hostpath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)

7. Downward API
   [https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/](https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/)

8. ConfigMaps
   [https://kubernetes.io/docs/concepts/configuration/configmap/](https://kubernetes.io/docs/concepts/configuration/configmap/)

9. Secrets
   [https://kubernetes.io/docs/concepts/configuration/secret/](https://kubernetes.io/docs/concepts/configuration/secret/)

10. Persistent Volumes
    [https://kubernetes.io/docs/concepts/storage/persistent-volumes/](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

11. Persistent Volume Claims
    [https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)

12. StorageClasses
    [https://kubernetes.io/docs/concepts/storage/storage-classes/](https://kubernetes.io/docs/concepts/storage/storage-classes/)

13. Volume Binding Mode
    [https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode](https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode)

14. Access Modes
    [https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)

15. Volume Modes
    [https://kubernetes.io/docs/concepts/storage/persistent-volumes/#volume-mode](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#volume-mode)

16. Reclaim Policy
    [https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaiming](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaiming)

17. Container Storage Interface (CSI)
    [https://kubernetes.io/docs/concepts/storage/volumes/#csi](https://kubernetes.io/docs/concepts/storage/volumes/#csi)

18. CSI Drivers (List)
    [https://kubernetes-csi.github.io/docs/drivers.html](https://kubernetes-csi.github.io/docs/drivers.html)

19. CRI, CNI, and CSI Interfaces Overview
    [https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/)

---
