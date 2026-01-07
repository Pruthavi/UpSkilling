# How to Debug / debugging issues in a Container or Pod

### [What Is CrashLoopBackOff?](https://kodekloud.com/blog/debug-crashloopbackoff/?utm_source=google&utm_medium=cpc&utm_campaign=KK_PMAX_IN_B2C-Cloud_CONV_NAN_PROS_MULT_NA-TB&utm_term=&utm_content=&utm_term=&utm_campaign=KK_PMAX_IN_B2C-Cloud_CONV_NAN_PROS_MULT_NA-TB&utm_source=adwords&utm_medium=ppc&hsa_acc=8039903603&hsa_cam=22329343884&hsa_grp=&hsa_ad=&hsa_src=x&hsa_tgt=&hsa_kw=&hsa_mt=&hsa_net=adwords&hsa_ver=3&gad_source=1&gad_campaignid=22329343167&gclid=Cj0KCQjwxdXBBhDEARIsAAUkP6j4Y4h9K_knniE0t8BGii8PBUG_d3ynA8Hs7zLhrkg_MuG9n5DdL_saAklCEALw_wcB#what-is-crashloopbackoff)

In Kubernetes, a Pod enters a state known as CrashLoopBackOff when an application inside the Pod keeps crashing repeatedly. ```CrashLoopBackOff``` **means the container in this Pod is failing and being restarted, over and over again.** Below are some factors that can get a Pod into a CrashLoopBackOff state:
- **Bugs within the application**
- **Insufficient resources**
- **Missing or incorrect configuration details**
- **Port conflicts**

---

### Debugging CrashLoopBackOff

1. Run ```kubectl describe pod <pod-name>``` :  to get detailed Pod information
    - check Exit code.
    - Exit code of 0 means that the process ran successfully without errors, while a **non-zero exit code indicates some sort of error**.
    - To gather more insights, scroll down to the Events section.
        - ```Back-off restarting failed container``` means that the kubelet attempted to restart the container after a failure but was unsuccessful. As a result, the kubelet will make another attempt to restart the container.
        - What does **"Back-off"** mean? Kubernetes, by default, restarts a Pod when it crashes. If the application doesn’t recover after the restart, Kubernetes doesn’t immediately attempt to restart the Pod again. Instead, it waits for some time. Why? Because it hopes the waiting time resolves any temporary issue the application may have. If the issue still persists, Kubernetes increases the waiting time between restart attempts by up to a certain limit. This approach is referred to as the back-off strategy. 
2. Run ```kubectl logs <POD-NAME>``` 
    - Pod logs are records of events within an application. They can often provide valuable insights into any issues the application is facing. 

---

### [How to Get Pod Logs in Kubernetes](https://kodekloud.com/blog/kubectl-logs/?_gl=1*w49eag*_up*MQ..*_gs*MQ..&gclid=Cj0KCQjwxdXBBhDEARIsAAUkP6j4Y4h9K_knniE0t8BGii8PBUG_d3ynA8Hs7zLhrkg_MuG9n5DdL_saAklCEALw_wcB)

1. Run ```kubectl attach <POD-NAME>``` : before we can see these logs in real time, we need to attach to the Pod. Attaching to the Pod essentially means that we're **connecting our terminal to the running container within the Pod**. This will allow us to observe the internal operations of the container as they happen.
    - Press ```CTRL + C``` to detach from the currently running container and let it run in the background.
2. Our terminal is now attached to the running container. Next, open a new terminal window.
3. Run ```kubectl exec -it <POD-NAME> -- /bin/bash``` : will initiate an interactive bash shell session.
4. Run ```kubectl logs -f <pod-name>``` : View Pod logs in real time. -f flag (short for "--follow")
5. Fetch a specific number of lines of Pod logs
    - Run ```kubectl logs --tail=5 <POD-NAME>``` : To retrieve the most recent five lines of the log generated.
6. View logs of an exited container
    - The kubectl logs command retrieves logs only from the currently running container.
    - Run ```kubectl logs –tail=10 <POD-NAME> -p``` If a container had crashed and was replaced, you can still retrieve the logs from its previous instance using the ```-p (or --previous)``` flag. This can be particularly useful when investigating the root cause of a container crash.
7. Fetch Pod logs from a specific time period
    - Run ```kubectl logs --since=1h <POD-NAME>``` : to fetch logs produced by the container within the last hour. 
        - ```-since``` flag accepts a time duration as its argument. 
        - For instance, to fetch logs from the last 1 hour, 30 minutes, and 20 seconds, you would use ```--since=1h30m20s```.
    - Run ```kubectl logs --since-time=’2023-05-14T00:00:00Z’ <POD-NAME>``` : to fetch logs starting from midnight on 14th May 2023
        - ```--since-time``` flag use to retrieve logs from a specific date and time onwards.
        - ```YYYY-MM-DDTHH:MM:SSZ``` This flag requires an argument.
            - **YYYY** represents the year
            - **MM** represents the month
            - **DD** represents the day
            - **T** is a separator indicating the start of the time portion
            - **HH** represents the hour
            - **MM** represents the minute
            - **SS** represents the second
            - **Z** signifies Coordinated Universal Time (UTC)
8. Fetch the logs of a specific container in a multi-container Pod
    - Run ```kubectl logs my-pod -c nginx``` : ```-c``` flag is followed by the name of the specific container ("nginx") from which we want to retrieve the logs.
    - When dealing with a multi-container Pod, you can fetch the logs of a specific container by specifying its name using the ```-c``` flag. 

---

### [How to Execute Shell Commands Into a Container](https://kodekloud.com/blog/kubectl-exec/?_gl=1*hiylho*_up*MQ..*_gs*MQ..&gclid=Cj0KCQjwxdXBBhDEARIsAAUkP6j4Y4h9K_knniE0t8BGii8PBUG_d3ynA8Hs7zLhrkg_MuG9n5DdL_saAklCEALw_wcB)

Kubectl Exec Syntax ```kubectl exec [OPTIONS] POD_NAME -- COMMAND [ARGS...]```
- **--**: This is a separator that tells "kubectl exec" to treat all subsequent arguments as the command to execute inside the container.
- **[ARGS...]**: These are optional arguments to the command you want to execute.
- **-i** flag stands for "interactive"
- **-t** flag is used to allocate a pseudo-TTY (terminal) means terminal session with the container.

---

### [How to Restart a Pod in Kubernetes](https://kodekloud.com/blog/kubernetes-pod-restart/?_gl=1*13b95qq*_up*MQ..*_gs*MQ..&gclid=Cj0KCQjwxdXBBhDEARIsAAUkP6j4Y4h9K_knniE0t8BGii8PBUG_d3ynA8Hs7zLhrkg_MuG9n5DdL_saAklCEALw_wcB)

>When we say a Pod is "restarted," it usually means a Pod is deleted, and a new one is created to replace it. The new Pod runs the same container(s) as the one that was deleted.

1. Understanding Kubernetes Pod Restart Policy
    - In Kubernetes, a Deployment manages the lifecycle of one or more Pods. When we define a Deployment using a YAML file, the spec field of the Pod template contains the configuration for the containers running inside the Pod. The restartPolicy field is one of the configuration options available in the spec field. It allows you to control how the Pods hosting the containers are restarted in case of failure.
    - set the restartPolicy field to one of the following three values: 
        - **Always**: Always restart the Pod when it terminates.
        - **OnFailure**: Restart the Pod only when it terminates with failure.
        - **Never**: Never restart the Pod after it terminates.
    - **Note** that if you don’t explicitly specify the restartPolicy field in a Deployment configuration file, Kubernetes sets the restartPolicy to Always by default.
2. Methods to Restart Kubernetes Pod
    1. Run ```kubectl delete pod <POD-NAME>``` : Deleting the Pod
    2. Modifying the fields in ```spec.template.spec.containers```
        - In a Deployment configuration file, the spec.template.spec.containers field describes the container(s) that should exist in a Pod overseen by the Deployment. This field holds various specifications about the container, such as its image, any necessary environment variables, and any required volumes, among others. Any modification to the value of these fields effectively changes the Pods' definition, triggering a Pod restart.
        - ```kubectl set image deployment/demo-deployment alpine-container=alpine:3.16``` To see the Pod restart in action, let’s change the alpine image version from 3.15 to 3.16
        - During Pod rollout process, Kubernetes creates Pods with the updated image and gradually phases out the old Pods.
        - Depending on your configuration file, you could modify other fields such as the ```env``` entries, ```volumeMounts```, and ```resources``` fields to trigger Pod restart.
    3. Using the "kubectl rollout restart" command
        - Run ```kubectl rollout restart deployment/demo-deployment``` : restart a Pod, without making any modifications to the Deployment configuration.
3. How Do I Restart Kubernetes Pod Without Downtime?
    - The Deployment resource in **Kubernetes has a default rolling update strategy, which allows for restarting Pods without causing downtime**. Here's how it works: Kubernetes gradually replaces the old Pods with the new version, minimizing the impact on users and ensuring the system remains available throughout the update process.
    - To restart a Pod without downtime, you can choose between two methods: using a Deployment (2) or using the ```kubectl rollout restart``` command.
    - Run ```kubectl describe deployment/demo-deployment``` to confirm that Kubernetes uses a rolling update strategy by fetching the Deployment details.
    - The RollingUpdateStrategy field has a default value of 25% max unavailable, 25% max surge. **25% max unavailable** means that during a rolling update, 25% of the total number of Pods can be unavailable. And **25% max surge** means that the total number of Pods can temporarily exceed the desired count by up to 25% to ensure that the application is available as old Pods are brought down.

---
### [How Kubernetes Works with Docker](https://kodekloud.com/blog/demystifying-container-orchestration-how-kubernetes-works-with-docker/?_gl=1*edqn1r*_up*MQ..*_gs*MQ..&gclid=Cj0KCQjwxdXBBhDEARIsAAUkP6j4Y4h9K_knniE0t8BGii8PBUG_d3ynA8Hs7zLhrkg_MuG9n5DdL_saAklCEALw_wcB)
**Containers** provide a lightweight and isolated runtime that ensures applications inside them run consistently across different environments.

**Container orchestration** is the process of automating the deployment, management, and coordination of containers across multiple hosts or clusters.

**Docker** lets you bundle an application and everything it needs into a portable unit, known as a container, that can run anywhere. This solves the issue of having different results on different machines and makes sure everything works the same everywhere. Key Docker Components:
1. **Docker images**: This is a small, self-contained, and runnable software bundle that has everything required to run an application, such as the code, runtime, system tool, and libraries. Images are built from a series of instructions specified in a Dockerfile, which defines the environment and dependencies of the application.
2. **Docker containers**: A Docker container is an instance of a Docker image. It provides an isolated runtime environment for running the application. Containers are portable and can be easily moved between different environments, ensuring consistency and reproducibility. You can start, stop, restart, and remove containers using the Docker CLI or API. You can also attach to a running container and execute commands inside it.
3. **Docker registries**: Docker registries are repositories where Docker images are stored and distributed. The most popular Docker registry is Docker Hub, a public registry that hosts thousands of pre-built images. You can push and pull images from registries using the Docker CLI or API. You can also use private registries, such as Docker Trusted Registry, which is a secure and enterprise-grade registry solution.
4. **Docker Hub**: This is the world's largest online repository of Docker images. It allows you to browse, search, and download millions of images for various applications and use cases. It also allows you to create and manage your repositories, where you can store and share your images with others. You can also use Docker Hub to automate your image builds and tests as well as to scan your images for security vulnerabilities.
5. **Docker Scout**: A web-based tool that helps you monitor and troubleshoot your containers and applications. It allows you to view the status, logs, metrics, and events of your containers, as well as to set alerts and notifications for any performance issues. You can also use Docker Scout to perform root cause analysis and to get recommendations for resolving problems.
6. **Docker Extensions**: These are plugins that extend the functionality of Docker. They allow you to integrate Docker with other tools and services, such as logging, monitoring, networking, storage, and security tools. You can find and install Docker Extensions from the Docker Store, which is a curated marketplace of trusted and verified extensions.

### Key Kubernetes Components
1. **Pods**: They are the smallest and most basic units of computation in Kubernetes. A Pod is a group of one or more containers that share the same network and storage resources and are deployed on the same host. Pods are ephemeral, meaning that they can be created and destroyed at any time. They are usually managed by higher-level controllers, such as deployments, which ensure that their desired number and state are maintained.
2. **Nodes**: These are the physical or virtual machines that run your Pods. Each node has a kubelet, which is an agent that communicates with the master node and manages the Pods on the node. Nodes also have other components, such as a container runtime, a kube-proxy, which handles the network routing for the Pods, and a kube-DNS, which provides DNS services for the Pods.
3. **Master node**: The master node in a Kubernetes cluster oversees the entire cluster's operation and manages the scheduling and deployment of Pods. It coordinates communication between nodes and maintains the desired state of the cluster.
4. **Control plane**: The control plane is the brain of the Kubernetes cluster. It has the API server, scheduler, etcd, and controller manager, that handle the orchestration and management of the cluster.
    - **API server**: The API server is the main entry point for all the communications between the nodes and the control plane. It exposes the Kubernetes API, which allows you to interact with your cluster using the kubectl CLI, the Kubernetes dashboard, or other tools and clients.
    - **Scheduler**: The scheduler is responsible for assigning Pods to nodes based on the resource availability and requirements of the Pods.
    - **Controller manager**: The controller manager runs various controllers that monitor and manage the state of your cluster. For example, the replication controller ensures that the desired number of Pods are running for a given deployment, the service controller creates and updates the load balancers for your services, and the node controller handles the node registration and health checks.
    - **Etcd**: Etcd is a distributed key-value store that stores the configuration and state data of your cluster. It is used by the API server and the other control plane components to store and retrieve the cluster information.

### How Kubernetes Works Together with Docker
1. Containerization with Docker: Docker builds images using **Open Container Initiative (OCI)** image format.  The OCI image format is designed to be platform-agnostic, meaning that containers created using this format can be run on any platform that supports the OCI standard.
2. Role of Kubernetes in Managing Docker Containers: ==Docker focuses on the creation and packaging of containers, while Kubernetes takes on the responsibility of managing and orchestrating these containers at scale. Kubernetes provides a powerful set of features that simplify the deployment and management of containers, ensuring high availability, scalability, and fault tolerance.==
3. Container Orchestration Features of Kubernetes:
    - **Scaling**: Kubernetes allows you to scale your application by adding or removing Pods based on the workload. This ensures that your application can handle increased traffic while still maintaining optimal performance. Alternatively, you can use the <font color="cyan">Horizontal Pod Autoscaler (HPA) to automatically scale your pods, or the Vertical Pod Autoscaler (VPA) to automatically scale your pods’ resources, or the Cluster Autoscaler (CA) to automatically scale your nodes based on defined metrics</font>.
    - **Load balancing**: Kubernetes automatically distributes incoming network traffic among your Pods using services and ingresses. This helps prevent the overloading of some Pods.
    - **Service discovery**: Kubernetes provides a built-in DNS service, kube-DNS, that allows containers within the cluster to discover and communicate with each other using the Pod or service names. It allows you to use simple and human-readable names, such as my-service.my-namespace.svc.cluster.local, to access your services.
    - **Rolling updates**: Kubernetes supports rolling updates, allowing you to update your application without downtime. It works by gradually replacing old Pods with new ones, ensuring a smooth transition and minimizing any disruption to users. Use the Deployment object to automate and manage the rolling updates, using parameters such as replicas, maxUnavailable, and maxSurge.

### Benefits of Using Kubernetes with Docker
1. **Enhanced scalability**: Kubernetes enables you to scale applications containerized using Docker easily and efficiently using autoscaling, load balancing, and service discovery features. You can handle any amount of traffic and demand without compromising the performance of your application.
2. **Improved resource utilization**: Kubernetes optimizes resource utilization by efficiently scheduling and managing containers across nodes. It ensures that containers are placed on nodes with available resources, preventing resource bottlenecks and maximizing cluster efficiency. Kubernetes also allows for the optimization of resources utilized by Pods using horizontal or vertical autoscaling features.
3. **Automated deployment and management**: Kubernetes simplifies the deployment and management of containerized applications. It provides a declarative approach to defining the desired state of your applications, allowing Kubernetes to handle the complex tasks of deploying, scaling, and managing containers.

---

### [How to Fix ImagePullBackOff & ErrImagePull in Kubernetes](https://kodekloud.com/blog/fix-imagepullbackoff-errlimagepull-in-kubernetes/?_gl=1*7xnytq*_up*MQ..&gclid=Cj0KCQjwxdXBBhDEARIsAAUkP6j4Y4h9K_knniE0t8BGii8PBUG_d3ynA8Hs7zLhrkg_MuG9n5DdL_saAklCEALw_wcB)

### What Causes ImagePullBackOff & ErrImagePull Errors?
>The ImagePullBackOff and ErrImagePull errors are two of the most common pod failures in Kubernetes. They both **mean that the pod cannot start because the container image cannot be pulled from the registry**. The difference between them is that **ErrImagePull is the initial error**, and ImagePullBackOff is the subsequent error after Kubernetes retries to pull the image several times and fails.

Below are 5 possible causes of these errors.
- Cause 1: [Network Issues Preventing Image Pull](#network-issues)
- Cause 2: [Invalid Image Names or Tags](#invalid-image-names-or-tags)
- Cause 3: [Insufficient Storage or Disk Issues](#insufficient-storage-or-disk-issues)
- Cause 4: [Unauthorized Access](#unauthorized-access)
- Cause 5: [Image Registry Issues](#image-registry-issues)

#### Network Issues
>One of the possible causes of the ImagePullBackOff or ErrImagePull errors is network issues that prevent Pods and nodes from accessing the remote container image registries. This can be due to the following:
- The registry URL is incorrect or unreachable.
- The network or firewall configuration is blocking the connection to the registry.
- The proxy settings are not configured properly.

To troubleshoot this cause, do the following:
1. **Check network connectivity**: Validate that the Pods and nodes can access the remote container image registries by using the `curl` or `wget` commands.
2. **Check firewall rules**: If you have a network firewall, make sure that it allows outbound access to the required ports for the registry. For instance, if your registry is Docker Hub, you must connect to port 443 for HTTPS. You can use commands like `iptables` or  `firewall-cmd` to see and change the firewall rules on your nodes. If the firewall rules are not configured properly, you need to update them to allow the connection to the registry.
3. **Check proxy settings**: If you are pulling images through a proxy, make sure that you configure the HTTPS proxy settings on your nodes and Pods. You can use the `https_proxy` environment variable to set the proxy URL on your nodes and Pods. You can also use the `imagePullSecrets` field in your Pod spec to provide the proxy credentials to your Pods.

#### Invalid Image Names or Tags
>Another reason for these errors is the mismatch between the image names or tags used and the image names and tags in the registry. This might be because there are issues with image names or tags, like typos, mismatch with the registry or using the latest tag which might cause unexpected updates.

To fix this problem, do the following:
1. **Validate image names and tags**: Check that all the image names and tags in your Pod specs are correct and match the images in the registry. Use the `kubectl get pod` and `kubectl describe pod` commands to check the image names and tags in your Pod specs. If the image name or tag is incorrect, you need to fix it in your Pod spec and redeploy your Pod.
2. **Pull image manually**: Try pulling the image directly from the command line interface (CLI) to verify that the image name and tag are valid and exist in the registry. You can use the `docker pull` or `podman pull` commands to pull the image from the registry. If the command fails, You need to fix the image name or tag in your Pod spec or push the image to the registry if it does not exist.
3. **Check for misspelled names or tags**: To avoid this, you should use descriptive and consistent image names and tags. Additionally, avoid using the `latest` tag, which can cause unexpected image updates and inconsistencies.

#### Insufficient Storage or Disk Issues
>Due to this error on the nodes, it will prevent the image from being downloaded and stored. This occurs when:
- The node disk is full or has insufficient space for the image
- The node disk is slow or has high I/O latency
- The image is too large or has too many layers

To diagnose this as the potential root cause, you should:
1. **Check available storage capacity**: Pods may fail to start if there is insufficient disk space on the node to store the image. Run the `df -h` command to check the available storage capacity on the node.
2. **Check disk I/O performance**: Saturated disk I/O can cause image pull timeouts or failures, especially if the image is large or has many layers. Run the `iostat` command to check the disk I/O performance on the node. If the command shows that the disk I/O is high or has high latency, you need to improve the disk I/O performance by reducing the disk load, using faster disks, or optimizing the image size or layers.

#### Unauthorized Access
>This can happen because: 
- The registry requires authentication and the credentials are missing or invalid
- The service account does not have permission to pull the image
- The credentials are expired or revoked

To troubleshoot potential unauthorized access problem, you should:
1. **Validate image pull secret**: If the registry requires authentication, you need to provide the credentials to the Pod using an image pull secret. You can run the `kubectl create secret` command to create an image pull secret with the credentials and then use the `imagePullSecrets` field in your Pod spec to reference it.
2. **Ensure the service account has pull permissions**: You can run the following commands to check the role and role binding of the service account `my-service-account` in the `default` namespace.
```kubectl
# Check the role of the service account
   kubectl get role my-role -n default

# Check the role binding of the service account
   kubectl get rolebinding my-rolebinding -n default
```
3. **If the credentials are expired or revoked**, you need to renew them or create new ones and update the image pull secret or the service account accordingly.

#### Image Registry Issues
>This can happen due to the following reasons:
- The registry is down or unreachable
- The registry does not have the requested image or tag
- The registry has errors or bugs that affect the image pull

When you want to fix this problem, you should:
1. **Confirm registry is up and running**: Check the status and availability of the registry by using the `curl` or `wget` commands to test the registry URLs from your browser or CLI. If registry is down or unreachable, you will need to contact the registry provider or administrator to resolve the issue.
2. **Check the registry for requested images**: Check the registry for the existence of the requested images and tags by using the registry web interface or API. If the URL does not show the image and tag, it means that the registry does not have the requested image and tag. You need to push the image and tag to the registry or fix the image name and tag in your pod spec if they are incorrect.
3. **Trace the registry logs for errors**: Trace the logs on the registry for any errors or bugs that might affect the image pull. You can use the registry web interface or API to access the logs, or contact the registry provider or administrator to get the logs. If the logs show any errors or bugs, you will also need to contact the registry provider or administrator to resolve them.

### How to Avoid ImagePullBackOff & ErrImagePull Errors?
>Following the best practices below will help you avoid these errors:
- Use descriptive and consistent image names and tags, and avoid using the latest tag.
- Use a reliable and secure registry service, such as Docker Hub, Azure Container Registry, or Amazon Elastic Container Registry, and configure the registry’s URL correctly in your Pod spec.
- Use secrets to store and provide the registry credentials to your Pods, and avoid hard-coding the credentials in your Pod spec or Dockerfile.
- Test your images locally before pushing them to the registry, and make sure they are compatible with your Kubernetes cluster version and architecture.
- Monitor your network and firewall settings, and ensure that your nodes and Pods can communicate with the registry without any issues.
- Monitor your node disk space, and ensure that you have enough space for your images and Pods.

---

