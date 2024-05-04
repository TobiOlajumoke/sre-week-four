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

