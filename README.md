## aws-xray-kubernetes

Code examples showing how to run AWS X-Ray on a Kubernetes cluster for deep application insights. Please also see the acompanying [blog post](https://aws.amazon.com/de/blogs/compute/application-tracing-on-kubernetes-with-aws-x-ray/) for background information.

# Changelog

* **03/31/2020** Update deployment specs to latest k8s version. Use official AWS X-Ray Docker image.
* **03/17/2020** Update sample app dependencies  

## Run AWS X-Ray on Kubernetes
The xray-daemon folder contains the code required to build and deploy an AWS X-Ray daemon Docker image and deploy this to an existing EKS or Kubernetes cluster.

Utilize the buildspec.yml for AWS CodeBuild to build and push the Docker image to an ECR repository.

## Deploying

Set up the correct URI pointing to your ECR repository where the X-Ray image is stored in the xray-k8s-daemonset.yml file. Then run ```kubectl apply -f xray-k8s-daemonset.yaml``` to install the AWS X-Ray daemons on your Kubernetes cluster.

## Demo apps

Two simple demo applications are provided to showcase how AWS X-Ray enables deep application insights into a microservices architecture.

To install and run both services build both container images and upload them to ECR repositories. Edit the ```k8s-deploy.yml``` file in the ```demo-app``` directory and set up the correct repository URI's.

Afterwards simply run ```kubectl apply -f k8s-deploy.yml``` to install both services. Look up the endpoint for service-a and send a couple of requests against this endpoint. Switch to the AWS X-Ray console and see how the traces are showing up in the console.

## License

This project is licensed under the Apache 2.0 License.
