# kubernetes-gcp-service-account

This is a small Github repo which is meant to serve as reference to combine a GCP service account to some Kubernetes account permissions.
It also includes a container for running Google Cloud SDK and Kubectl directly inside

## Usage

1. Create a service account on GCP for running jobs
  * ```gcloud iam service-accounts create service-account --display-name service-account --project PROJECTHERE```
2. Create some credentials for new service account
  * ```gcloud iam service-accounts keys create --iam-account service-account@PROJECTHERE.iam.gserviceaccount.com ./build/gcloud-config```
3. Give service account Kubernetes viewer permissions
  * ```gcloud projects add-iam-policy-binding PROJECTHERE --member serviceAccount:service-account@PROJECTHERE.iam.gserviceaccount.com --role roles/container.clusterViewer```
4. Add service account to the bottom of `kubernetes_resources/deploy-user.yml```
5. Add cluster role and rolebinding for that user to the cluster
  * ```kubectl apply -f kubernetes_resources```
6. Build Docker, passing relevant cluster info `cd build && docker build . -t kubernetes-runner --build-arg CLUSTER=CLUSTERHERE --build-arg PROJECT=PROJECTHERE --build-arg ZONE=europe-west1-d`
8. Run container and get pods in specificed namespace
  * ```docker run kubernetes-runner:latest get pods -n example-namespace```
9. Alternatively run kubernetes directly inside the container, some examples below:
  * ```docker run -it --entrypoint /bin/sh kubernetes-runner:latest```
  * Then in the container to create a temporary pod
  * ```kubectl run -n example-namespace -it nettools --image=travelping/nettools --rm --restart=Never -- sh```
