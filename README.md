# Release engineering at UpCommerce

James, the Engineering Lead at UpCommerce, liked the SRE team's ideas about taking the payment system implementation out of UpCommerce's original implementation and turning it into a microservice. However, he is skeptical about the implementation details.  He has asked that the SRE team use a canary deployment strategy to implement the changes to the UpCommerce app. The details of the canary deployment are as follows:

- Deploy a canary version of the app on port 5003. 

- The canary deployment should have only 1 replica, since it will serve only a few customers.

- In the event that the canary fails, shut it down and rollback to the previous stable condition of UpCommerce's deployment before the canary release.

UpCommerce's DevOps team uses Helm as the tool of choice for release management. 

# Getting your development environment ready

For this week's project, you will use GitHub Codespaces as your development environment, just like you did for the projects in previous weeks.

## Steps

1. Create a fork of the week's repository.

2. When you have created a fork of the week's repository, start a codespace on the main branch.

3. Run the command below in your codespace's terminal to create a single-node, Kubernetes cluster using Minikube: 
```
minikube start
```

4. Once your Minikube cluster is running, enter the command below:
```
kubectl create namespace sre
```

This creates a namespace in your Kubernetes cluster named sre. It is within this namespace that you'll do all the tasks required for this project.

## Tasks

1. Deploy the helm chart using the command below: 
```
helm install upcommerce ./upcommerce -n sre
```

2. In the templates/ directory, create canary-deployment.yml and canary-service.yml manifest files and fill in the required Helm templates for them.  If you need some ideas, you can find Helm templates for the aforementioned manifests below:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-canary-app
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}-canary-app
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}-canary-app
    spec:
      containers:
        - name: canary
          image: {{ .Values.canary.image }}
          ports:
            - containerPort: 5003
          imagePullPolicy: {{ .Values.imagePullPolicy }}
          resources:
            limits:
              cpu: {{ .Values.cpuLimit }}
              memory: {{ .Values.memoryLimit }}


```
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-canary-service
spec:
  selector:
    app: {{ .Release.Name }}-canary-app
  ports:
    - protocol: TCP
      port: 5003
      targetPort: 5003
```

3. Update the values.yml file with the necessary values that will be cross-referenced in the canary templates (i.e. the image for the canary deployment) using the detail below:
```
canary:
  image: uonyeka/canary:linux-amd64
```
If you need a hint on what your values.yml file should look like after the update, click on the expandable section below:   +

```
replicaCount: 1
imagePullPolicy: Always
cpuLimit: "0.5"
memoryLimit: "2Gi"
upcommerce:
  image: uonyeka/upcommerce:v3
canary:
  image: uonyeka/canary:linux-amd64
  ```\

4. Run the command below to upgrade your Helm installation with the latest changes:

```
helm upgrade upcommerce ./upcommerce -n sre
```

5. Run the command below to show you details of all deployments in the sre namespace:
```
kubectl get deployment -n sre -o wide
```

6. Because the canary deployment is unstable, you have to roll back the Kubernetes deployment to the last stable state. You can run the command below to get the list of all Helm releases in your cluster and their release numbers
```
helm list -a -n sre 
```
or run the command below to get a historical overview of the Upcommerce release
```
helm history upcommerce -n sre
```

When you have gotten the release numbers, rollback to the stable release using the command below (replace <release_number> with the actual number of the release you want to roll back to):
```
helm rollback upcommerce <release_number> -n sre
```
To confirm that things have been rolled back, please run any one of the commands below:
```
helm list -a -n sre
helm history upcommerce -n sre
```

7.  Here are some other commands that you can use to better understand and explore your Helm deployment:
```
helm ls -n sre    # List all releases (deployments).
helm status upcommerce -n sre   # Get detailed information about a specific release.
helm uninstall upcommerce -n sre  # Uninstall a release.
helm history upcommerce -n sre  # View the revision history of a release.
helm show values ./upcommerce -n sre  # Display default values from the chart.
helm show readme ./upcommerce -n sre  # Show the chart’s README.
helm lint ./upcommerce -n sre  # Check your chart for issues.
helm template upcommerce ./upcommerce -n sre  # Render templates without installing.
```
