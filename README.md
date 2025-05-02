# k0rdent Demo Repo

## What this is for

Small project to create a Kubernetes Individual Development Platform for Chainguard demos and testing.

It includes scripts and examples for basic k0rdent usage.

## Table of Contents

1. [Setup](#setup)
   1. [Prerequisites](#prerequisites)
   1. [General setup](#general-setup)
   1. [Infrastructure setup](#infra-setup)
      1. [AWS setup](#aws-setup)
      1. [Azure setup](#azure-setup)
      1. [OpenStack setup](#openstack-setup)
1. [Standalone Cluster Deployment](#standalone-cluster-deployment)
1. [Cleaning up](#cleaning-up)

## Setup

### Prerequisites

Tools needed to run this demo are
- Docker
- Git
- make

If you're unsure about whether your system is supported and whether all tools are installed correctly, we provide a script that helps you answer this questions.
Follow these steps to download and run the script to check your setup.

1. Ensure you have a Bash-compatible shell (Linux, macOS).
1. Run the script
   ```shell
   ./scripts/check-prerequisites.sh
   ```

The Setup part is assumed to be run before an cluster are created.

To get the full list of commands run `make help`.

### General Setup

> Expected completion time ~10 min

> **Check Docker 'kind' network**  
> The default Docker network for `kind` sometimes conflicts if it is created with a `172.18.0.0/16` subnet.  
> To avoid issues, remove or recreate the `kind` network with a different subnet **before** bootstrapping your cluster:
>
> ```shell
> # 1. Check if the "kind" network already exists
> docker network inspect kind
>
> # 2. If it exists, check the subnet
> docker network inspect kind --format '{{(index .IPAM.Config 0).Subnet}}'
>
> # 3. If the output is 172.18.0.0/16 (or any other conflicting subnet), remove the network:
> docker network rm kind
>
> # 4. Create or recreate it with a custom subnet (e.g., 10.24.0.0/16)
> docker network create kind --subnet=10.24.0.0/16
> ```
>
> Once the `kind` network is ready, continue with the [bootstrap-kind-cluster](#) step.

1. Create a k0rdent Management cluster with kind:
    ```shell
    make bootstrap-kind-cluster
    ```
    You could give it another name by specifying the `KIND_CLUSTER_NAME` environment variable. 

2. Install k0rdent into kind cluster:
    ```shell
    make deploy-k0rdent
    ```
    The Demos in this repo require at least k0rdent v0.3.0 or newer. You can change the version by specifying the `KCM_VERSION` environment variable. List of releases can be found [here](https://github.com/K0rdent/kcm/releases).

3. Monitor the installation of k0rdent:
    ```shell
    make watch-k0rdent-deployment
    ```
    In this command we track the `Management` object that is created by k0rdent. Don't worry if you get message that the object is not found, it can take some time.
    Wait until the output of the command be as follows to make sure that k0rdent project is fully installed:
    ```
    Status of the k0rdent components installation: 
    capi: true
    cluster-api-provider-aws: true
    cluster-api-provider-azure: true
    cluster-api-provider-openstack: true
    cluster-api-provider-vsphere: true
    k0smotron: true
    kcm: true
    projectsveltos: true
    ```
4. **Important!** If you are going to run demos in corporate or shared cloud accounts, and it is possible that someone else is running the same demos, you may end up in a situation where cloud resources with the same name already exist and you will get errors when deploying clusters. To avoid this, you can set an environment variable with your username. It can be any value though but it's required to be unique:
    ```
    export USERNAME=<your_username>
    ```


### Infra Setup

As next you need to decide into which infrastructure you would like to install the Demo clusters. This Repo has support for the following Infra Providers (more to follow in the future):

- AWS
- Azure
- GCP

> **Note on Cloud Provider Commands**  
> Throughout these demos, you might see commands referencing `aws`. For example:
> ```shell
> make apply-clustertemplate-demo-aws-standalone-cp-0.0.1
> ```
> If you are using **Azure** or **GCP** instead, simply replace `aws` with your provider name, for example:
> ```shell
> # Azure
> make apply-clustertemplate-demo-azure-standalone-cp-0.0.1
>
> # GCP
> make apply-clustertemplate-demo-gcp-standalone-cp-0.0.1
> ```
> The rest of the steps remain the same—just ensure you use the relevant commands for your chosen infrastructure.

#### AWS Setup

> Expected completion time ~2 min

This assumes that you already have configured the required [AWS IAM Roles](https://docs.k0rdent.io/v0.1.0/quickstart-2-aws/#attach-iam-policies-to-the-k0rdent-user) and have an [AWS account with the required permissions](https://docs.k0rdent.io/v0.1.0/quickstart-2-aws/#create-the-k0rdent-aws-user). If not follow the k0rdent documentation steps for them.

1. Export AWS Keys as environment variables:
    ```shell
    export AWS_ACCESS_KEY_ID="AWS Access Key ID"
    export AWS_SECRET_ACCESS_KEY="AWS Secret Access Key"
    ````
2. If you use SSO authentication in AWS, export session token:
    ```shell
    export AWS_SESSION_TOKEN="AWS Session Token"
    ```
2. By default, it will provision all resources in the `us-west-2` AWS region. If you want to change this, export `AWS_REGION` environment variable:
    ```shell
    export AWS_REGION="us-east-1"
    ```

3. Install Credentials into k0rdent:
    ```shell
    make apply-aws-creds
    ```

4. Check that credentials are ready to use
    ```shell
    make get-creds-aws
    ```
    The output should be similar to:
    ```
    NAME                        READY   DESCRIPTION
    aws-cluster-identity-cred   true    Basic AWS credentials
    ```

#### Azure Setup

> Expected completion time ~2 min

This assumes that you already have configured the required [Azure providers](https://docs.k0rdent.io/v0.1.0/quickstart-2-aws/#attach-iam-policies-to-the-k0rdent-user) and created a [Azure Service Principal](https://docs.k0rdent.io/v0.1.0/quickstart-2-aws/#attach-iam-policies-to-the-k0rdent-user). If not follow the k0rdent documentation steps for them.

1. Export Azure Service Principal keys as environment variables:
    ```
    export AZURE_SP_PASSWORD=<Service Principal password>
    export AZURE_SP_APP_ID=<Service Principal App ID>
    export AZURE_SP_TENANT_ID=<Service Principal Tenant ID>
    export AZURE_SUBSCRIPTION_ID=<Azure's subscription ID>
    ```

2. Install Credentials into k0rdent:
    ```
    make apply-azure-creds
    ```

3. Check that credentials are ready to use
    ```shell
    make get-creds-azure
    ```
    The output should be similar to:
    ```
    NAME                          READY   DESCRIPTION
    azure-cluster-identity-cred   true    Azure credentials
    ```

#### GCP Setup

> Expected completion time ~2 min

This assumes that you already have configured a Service Account in GCP, the instance type "n1-standard-2" is accessible, and an image called "projects/ubuntu-os-cloud/global/images/ubuntu-2004-focal-v20250213" is present.

1. Export Service Account Credential as environment variables:
    ```shell
    export GCP_CREDENTIAL="base64-encoded GCP credentials"
    ````
2. Install Credentials into k0rdent:
    ```shell
    make apply-gcp-creds
    ```

3. Check that credentials are ready to use
    ```shell
    make get-creds-gcp
    ```
    The output should be similar to:
    ```
    NAME                        READY   DESCRIPTION
    gcp-cluster-identity-cred   true    GCP credentials
    ```

## Standalone Cluster Deployment

> Expected completion time ~10-15 min

This shows how a simple standalone cluster from a ClusterTemplate can be created in the `kcm-system` namespace. It does not require any additional users in k8s or namespaces to be installed.

1. Install Test Cluster:
    ```shell
    make apply-cluster-deployment-aws-test1-0.0.1
    ```
    This will create an objects of type `ClusterDeployment` with very simple defaults from the ClusterTemplate `demo-aws-standalone-cp-0.0.1`.
    The yaml for this can be found under [`clusterDeployments/aws/0.0.1.yaml`](./clusterDeployments/aws/0.0.1.yaml) and could be modified if needed.
    The Make command also shows the actual yaml that is created for an easier demo experience.

    Available clusters are
    ```shell
    make apply-cluster-deployment-aws-test1-0.0.1 # AWS Standalone
    make apply-cluster-deployment-azure-test1-0.0.1 # Azure Standalone
    make apply-cluster-deployment-gcp-test1-0.0.1 # GCP Standalone
    make apply-cluster-deployment-eks-test1-0.0.1 # EKS
    make apply-cluster-deployment-ekscg-test1-0.0.1 # EKS with Chainguard VM
    ```

2. Monitor the deployment of the cluster and wait for it to be in Ready state:
    ```shell
    make watch-aws-test1
    ```

    Example of the output of fully deployed first cluster:
    ```
    NAME                READY   STATUS
    k0rdent-aws-test1   True    ClusterDeployment is ready
    ```

3. Create Kubeconfig for Cluster:
    ```shell
    make get-kubeconfig-aws-test1
    ```
    This will put kubeconfig for a cluster admin under the folder `kubeconfigs`.


4. Access Clusters through kubectl
    ```shell
    KUBECONFIG="kubeconfigs/kcm-system-aws-test1.kubeconfig" PATH=$PATH:./bin kubectl get node
    ```

    Example output (username suffix will be present only if you specified the `USERNAME` variable at the [`General Setup`](#general-setup) step):
    ```
    NAME                               STATUS   ROLES           AGE   VERSION
    k0rdent-aws-test1-<username>-cp-0             Ready    control-plane   19m   v1.31.2+k0s
    k0rdent-aws-test1-<username>-md-j87z9-fljb4   Ready    <none>          17m   v1.31.2+k0s
    k0rdent-aws-test1-<username>-md-j87z9-r85gs   Ready    <none>          17m   v1.31.2+k0s
    ```

## Cleaning up

As running the whole k0rdent setup can be quite taxing on your hardware, run the following command to clean up everything (both the public cloud resources mentioned above but also all local containers):
```shell
make cleanup
```

To reset management cluster and cleanup only `ClusterDeployment` objects you can run the command:
```shell
make cleanup-clusters
```