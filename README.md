## aws-xray-kubernetes

Code examples showing how to run AWS X-Ray on a Kubernetes cluster for deep application insights. Please also see the acompanying [blog post](https://aws.amazon.com/de/blogs/compute/application-tracing-on-kubernetes-with-aws-x-ray/) for background information.

# Changelog

* **04/05/2020** Clean up permissions for X-Ray daemon. Expose tcp and udp port.
* **03/31/2020** Update deployment specs to latest k8s version. Use official AWS X-Ray Docker image.
* **03/17/2020** Update sample app dependencies  

## Run AWS X-Ray on Kubernetes
The xray-daemon folder contains the code required to build and deploy an AWS X-Ray daemon Docker image and deploy this to an existing EKS or Kubernetes cluster.

Images have been built and pushed to Docker Hub for easier consumption of this project.

## Deploying

Set up the correct permissions in AWS IAM so your pod can utilize. Utilize the EKS feature for IAM for Service Accounts. See the file ``xray-k8s-daemonset.yaml`` or [EKS Userguide](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) for setup instructions. Then run ```kubectl apply -f xray-k8s-daemonset.yaml``` to install the AWS X-Ray daemons on your Kubernetes cluster.

Fallback option is to attache the IAM policy named ``arn:aws:iam::aws:policy/AWSXRayDaemonWriteAccess`` to the worker nodes in your cluster.

## Demo apps

Two simple demo applications are provided to showcase how AWS X-Ray enables deep application insights into a microservices architecture.

Simply run ```kubectl apply -f k8s-deploy.yml``` to install both services. Look up the endpoint for service-a and send a couple of requests against this endpoint. Switch to the AWS X-Ray console and see how the traces are showing up in the console.

## License

This project is licensed under the Apache 2.0 License.
