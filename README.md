# crossplane-training


## Requiremnts
1) GKE cluster with version k8s 1.22 or above
2) gcloud & kubectl setup with admin role
3) Chose your favorite IDE and enjoy the lab ^_^

## Overview 
[Crossplane](https://crossplane.io/) provides Declarative approach to infrastructure configuration and infrastructure management  that can be used to provision and manage any infrastructure on top of the K8s API.

## Crossplane vs Terraform
Crossplane often compared to hashicorp Terraform。 for the enterprise platform team, when Terraform when they can't meet the needs and look for alternatives, they usually find Crossplane， so there are similarities between the two open source projects. ：

* Both allow engineers to model their infrastructure as a declarative configuration
* Both support the use of “ provider ” plug-ins manage a large number of different infrastructures
* both are open source tools with strong communities.

Here a great [article](https://www.codestudyblog.com/8ten8/80328170911.html) to know more about the difference between crossplane and terraform.

## Install & Configure

### Configure and install

#### **Install crossplane on gke with helm**
Start with a Self-Hosted Crossplane

```
kubectl create namespace crossplane-system

helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update

helm install crossplane --namespace crossplane-system crossplane-stable/crossplane

```

Check Crossplane Status
```
helm list -n crossplane-system

kubectl get all -n crossplane-system
```

##### **Install CLI**

```
curl -sL https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh | sh
```

##### **Install GCP Provider**
Crossplane needs a provider configuration in order to be able to create resources in that specific provider (behind the scene it communicate with provider API).
Crossplane community Already made a getting started `Configuration`

```
kubectl crossplane install configuration registry.upbound.io/xp/getting-started-with-gcp:latest
```
Wait until all packages become healthy:
```
kubectl get pkg --watch
```

Get GCP Account Keyfile

```
# replace this with your own gcp project id and the name of the service account
# that will be created.
PROJECT_ID=my-project
NEW_SA_NAME=test-service-account-name

# create service account
SA="${NEW_SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"
gcloud iam service-accounts create $NEW_SA_NAME --project $PROJECT_ID

# enable cloud API
SERVICE="sqladmin.googleapis.com"
gcloud services enable $SERVICE --project $PROJECT_ID

# grant access to cloud API
ROLE="roles/cloudsql.admin"
gcloud projects add-iam-policy-binding --role="$ROLE" $PROJECT_ID --member "serviceAccount:$SA"

# create service account keyfile
gcloud iam service-accounts keys create creds.json --project $PROJECT_ID --iam-account $SA
```

Create a Provider Secret
```
kubectl create secret generic gcp-creds -n crossplane-system --from-file=creds=./creds.json
```

We will create the following ProviderConfig object to configure credentials for GCP Provider:
```
# replace this with your own gcp project id
PROJECT_ID=my-project
echo "apiVersion: gcp.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  projectID: ${PROJECT_ID}
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: gcp-creds
      key: creds" | kubectl apply -f -
```

Now, that you have configured crossplane to use GCP provider.
For example, you can check the available custom resource GkeCluster that is being created.

```
$ kubectl explain clusters --api-version=container.gcp.crossplane.io/v1beta2

KIND:     Cluster
VERSION:  container.gcp.crossplane.io/v1beta2

DESCRIPTION:
     A Cluster is a managed resource that represents a Google Kubernetes Engine
     cluster.

FIELDS:
   apiVersion   <string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind <string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata     <Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec <Object> -required-
     A ClusterSpec defines the desired state of a Cluster.

   status       <Object>
     A ClusterStatus represents the observed state of a Cluster.
```
