REST to SOAP Transformation with Apache Camel and Kong
==========================================

This example demonstrates how to use Apache Camel to frontend an exisiting legacy SOAP service with a RESTFUL interface.  This can be deployed to any flavor of Kubernetes, in this case, GKE.

## Prerequisites

1. Java 11+
2. Maven 3.8+
3. A running GKE instace

## Build

You can build this example using

```
    mvn package
```

## Run the example

Using the shell:

 1. Start the springboot service:

```
  $ mvn spring-boot:run
```

## Test the example:

Using Insomnia, import the Swagger spec using the following url:

```
     http://localhost:8080/camel/api-doc
```

You can test the transformation using any number you like in the URL:

```bash
curl -X GET "http://localhost:8080/camel/numbertowords/737373" -H "accept: application/json"
```

     
## Deploying to Kubernetes

To deploy to Google Cloud (GKE), we need to follow these steps.

## Prerequisites

1. Java 11+
2. Docker
3. DockerHub account
4. A Google Cloud account, with
5. gcloud CLI
6. kubectl

## Method

1. Assuming you have already provisioned gke, setup `kubectl` to point to your new gke cluster and test:

```bash
gcloud container clusters get-credentials <cluster name> --zone us-central1-a --project <project name>
kubectl get all
```

2.  Via the CLI (`/kong-soap-rest-transformation` directory), run the following commands to build the Docker image locally:

```bash
docker login
mvn clean spring-boot:build-image
```

3. Login to your public DockerHub repo (using the push commands popup), then tag the docker image and push it to the the repo using your correct repo ID:

```bash
docker tag docker.io/library/kong-rest-soap-transformation:1.0.0 simongreen02/calypso:kong-rest-soap-transformation
docker push simongreen02/calypso:kong-rest-soap-transformation
```

4. Update the repo ID in `src/k8s/deployment.yaml` to point to the correct ECR repo:

```yaml
    spec:
      containers:
      - image: simongreen02/calypso:kong-rest-soap-transformation
        name: kong-rest-soap-transformation
        resources: {}
status: {}
```

5. Deploy the image and service to gke:

```bash
kubectl apply -f src/k8s/deployment.yaml 
```

6. Run the following command and copy the `LoadBalancer Ingress` IP (or hostname):

```bash
kubectl describe service kong-rest-soap-transformation
```

7.   Using the above hostname, test the service with the following command:

```bash
curl http://35.226.32.108/camel/api-doc
```

8.  You can import the above URL into Insomnia and test the endpoint using the OpenAPI spec.

You can test the transformation using any number you like in the URL:

```bash
curl -X GET "http://35.226.32.108:8080/camel/numbertowords/122" -H "accept: application/json"
```

9.  If the request is successful, you should receive the following response:

```json
{
  "item": "one hundred and twenty two "
}
```