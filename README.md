# Kuberneties_k8s

## Deployment — adding files into containers

When you need files to be present in containers running inside a Pod, remember Kubernetes operates at the Pod level, not the individual container level. Making ad-hoc changes inside running containers is possible for debugging, but not reliable or reproducible for production. Recommended best practices:

- **Build a custom image**: Create a `Dockerfile` that adds the required files during image build. This is the most robust approach—bake files into the image so every Pod created from that image has the files already.
- **Push to a central registry**: Push your image to a registry (for example Docker Hub, GitHub Container Registry, or a private registry) and reference that image in your `Deployment` manifest. Tag images (semantic versions) to make rollbacks and CI/CD integration simple.
- **Automate builds**: Use CI to build, test, and push images. This avoids manual steps and ensures immutability and reproducibility.

Alternatives (useful for specific cases):

- **ConfigMaps / Secrets / Volumes**: For configuration or small text files, mount a `ConfigMap`, `Secret`, or a volume into the Pod.
- **Init containers**: Use an init container to download or generate files before the main container starts.
- **kubectl cp / exec**: These are useful for debugging, but changes are ephemeral and not suitable as a deployment strategy.

Why prefer images: baking files into the image ensures consistency across environments, simplifies scaling, and eliminates the need to modify running containers manually. Keep images minimal, use multi-stage builds where appropriate, and scan images for vulnerabilities as part of your pipeline.

Example workflow summary:

1. Add required files in a `Dockerfile`.
2. Build and tag the image locally or via CI: `docker build -t myrepo/myapp:1.2.3 .`
3. Push the image to the registry: `docker push myrepo/myapp:1.2.3`
4. Update your `Deployment` to use `myrepo/myapp:1.2.3` and apply it to the cluster.

This approach minimizes manual work and makes deployments predictable and auditable.

## Kubernetes workload controllers

### ReplicaSet
- What it is: A controller that ensures a specified number of identical Pod replicas are running at all times.
- Where it is used: Commonly used by `Deployment` objects to manage scaling and rolling updates of stateless applications.
- Why it is different: `ReplicaSet` focuses on keeping Pod replica counts stable, but it does not manage deployment strategies or rollbacks; `Deployment` adds those higher-level features.

### ReplicationController
- What it is: An older controller that also keeps a fixed number of Pod replicas running.
- Where it is used: Mostly legacy clusters or older examples. It has been superseded by `ReplicaSet`.
- Why it is different: It uses exact label matching and lacks the newer features of `ReplicaSet` and `Deployment`, so it is rarely used for new workloads.

### DaemonSet
- What it is: A controller that ensures one Pod copy runs on each selected node (or a subset of nodes).
- Where it is used: Ideal for node-level services such as log collectors, monitoring agents, security tools, or storage drivers.
- Why it is different: Unlike `ReplicaSet` or `Deployment`, it is not about scaling a fixed number of replicas; it is about deploying a Pod per node for cluster-wide coverage.

### StatefulSet
- What it is: A controller for stateful applications that need stable network IDs and persistent storage.
- Where it is used: Databases, message queues, and other stateful services requiring ordered startup, stable Pod identities, and persistent volumes.
- Why it is different: It preserves the identity of each Pod and manages storage lifecycle, unlike stateless controllers such as `ReplicaSet` and `Deployment`.
