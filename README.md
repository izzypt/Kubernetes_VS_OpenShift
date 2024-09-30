# Kubernetes_VS_OpenShift

Adapting a Kubernetes setup to OpenShift mainly involves adjusting the manifests to account for the differences in how OpenShift operates compared to vanilla Kubernetes. While OpenShift is built on top of Kubernetes, it adds several features and imposes security constraints that require changes in resource definitions.

Here are the key differences in terms of manifests and their adaptation:

### 1. **Security Context and User Permissions**
   - **OpenShift’s stricter security model**: OpenShift enforces the use of non-root containers by default, whereas Kubernetes does not enforce such restrictions. In OpenShift, containers run with a randomized user ID (UID) to ensure enhanced security.
     - **Kubernetes**: You may have a pod running as root (`runAsUser: 0`).
     - **OpenShift**: You need to remove any explicit root user or `runAsUser` specification unless absolutely necessary, or assign a `SecurityContextConstraints (SCC)` that allows privileged containers.
     - Example modification:
       ```yaml
       securityContext:
         runAsUser: 1001  # In OpenShift, this should be omitted or set to a non-root user
       ```

### 2. **Service Types**
   - **Route in OpenShift**: OpenShift uses `Route` objects to expose services to the outside world, whereas Kubernetes uses `Ingress`. The `Route` resource is specific to OpenShift.
     - **Kubernetes**: Uses `Ingress` with an ingress controller.
     - **OpenShift**: Replace `Ingress` with `Route`.
     - Example of a Route definition in OpenShift:
       ```yaml
       apiVersion: route.openshift.io/v1
       kind: Route
       metadata:
         name: my-route
       spec:
         to:
           kind: Service
           name: my-service
         port:
           targetPort: 8080
       ```

### 3. **Image Streams**
   - **OpenShift Image Streams**: OpenShift supports `ImageStreams`, which provide versioned references to container images and facilitate image management.
     - **Kubernetes**: Pulls container images directly from DockerHub or a private registry.
     - **OpenShift**: You might use `ImageStreams` instead of pulling images directly in the deployment manifest.
     - Example:
       ```yaml
       apiVersion: image.openshift.io/v1
       kind: ImageStream
       metadata:
         name: my-app
       ```

### 4. **SecurityContextConstraints (SCC)**
   - **OpenShift SCC**: OpenShift uses `SecurityContextConstraints` (SCC) to define permissions for running pods, whereas Kubernetes uses `PodSecurityPolicies` (PSP). SCCs are more restrictive and require specific settings.
     - In OpenShift, you may need to assign your deployment an SCC if it needs special privileges (e.g., `privileged`, `anyuid`, etc.).
     - Example:
       ```yaml
       kind: SecurityContextConstraints
       apiVersion: security.openshift.io/v1
       metadata:
         name: custom-scc
       runAsUser:
         type: RunAsAny
       ```

### 5. **Persistent Storage**
   - **OpenShift Persistent Volumes**: OpenShift supports persistent volumes similar to Kubernetes, but it includes additional storage classes and plugins.
     - You may need to review the storage class in OpenShift to ensure compatibility.
     - Example:
       ```yaml
       apiVersion: v1
       kind: PersistentVolumeClaim
       metadata:
         name: my-pvc
       spec:
         storageClassName: nfs
         accessModes:
           - ReadWriteOnce
         resources:
           requests:
             storage: 1Gi
       ```

### 6. **Pod Specifications and Annotations**
   - **Annotations for OpenShift features**: You may need to add specific annotations in your manifests to take advantage of OpenShift features (like auto-scaling, logging, or monitoring).
     - Example annotation for OpenShift monitoring:
       ```yaml
       metadata:
         annotations:
           openshift.io/monitoring: 'true'
       ```

### 7. **Network Policies**
   - **OpenShift SDN**: OpenShift uses its own SDN (Software-Defined Networking) by default. You may need to adapt your network policies, as OpenShift has different defaults for network isolation.
     - Example:
       ```yaml
       apiVersion: networking.k8s.io/v1
       kind: NetworkPolicy
       metadata:
         name: allow-frontend
       spec:
         podSelector:
           matchLabels:
             role: frontend
         policyTypes:
         - Ingress
       ```

### 8. **RBAC and OAuth**
   - **OpenShift OAuth integration**: OpenShift integrates with OAuth and has a slightly different way of handling Role-Based Access Control (RBAC). You may need to update your RBAC rules, especially if you are integrating with OpenShift’s authentication.

### 9. **Deployment Strategies**
   - **Rolling Deployment with OpenShift**: While OpenShift supports Kubernetes-style `Deployments`, it also has its own `DeploymentConfig`, which offers more control over deployment strategies (like custom hooks).
     - If your setup uses `DeploymentConfig`, you need to ensure the Helm chart is compatible with both Kubernetes `Deployments` and OpenShift `DeploymentConfig`.
     - Example `DeploymentConfig`:
       ```yaml
       apiVersion: apps.openshift.io/v1
       kind: DeploymentConfig
       metadata:
         name: my-app
       spec:
         replicas: 3
         selector:
           app: my-app
         template:
           metadata:
             labels:
               app: my-app
           spec:
             containers:
             - name: my-container
               image: my-image
       ```

### Summary of Manifest Changes:
1. Remove or modify any `runAsUser` or root-level permissions to comply with OpenShift's security model.
2. Replace Kubernetes `Ingress` with OpenShift `Route`.
3. Use `ImageStreams` for image management if needed.
4. Define `SecurityContextConstraints` if your pods need elevated privileges.
5. Ensure storage classes and network policies are compatible with OpenShift.

By making these adaptations, you can transition from a Kubernetes environment to OpenShift more smoothly while taking advantage of OpenShift-specific features and security enhancements.
