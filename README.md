# Ghost-helm

This is a proof of concept for the customer Drone Shuttles Ltd to launch their website using Ghost
The repo will help you to deploy a website using Ghost into an AWS Managed Kubernetes cluster(EKS). 
There are other repos which will help in automating the deployment of the whole infrastructure used for the website
- [eks-tf](https://gitlab.com/nord-cloud-ghost/eks-tf) - Terraform(IaC) code base to deploy EKS cluster 
- [eks-addons](https://gitlab.com/nord-cloud-ghost/eks-addons) - Helm chart code base to deploy addons on EKS cluster for monitoring and visualising Ghost app and its environments
#TODO:: Fix me with url
- [ghost-infra]() - Terraform code base to deploy infra related to Ghost application

## Infrastructure
To support development efforts three different environments (Dev, Staging, Prod) will be created in AWS and each environment mainly consists of the following:

- VPC (with private and public subnet) - Networking
- EKS cluster - Compute
- Route53 records - DNS service
- Lambda function - To support customer with serverless function to delete all posts
- Gitlab - For storing code and CI/CD
- S3 buckets - For storing terraform state files and creating backups for eks cluster
- IAM - For user and access management

All the environments will be managed by three different accounts within AWS for separation of concerns.

All EKS clusters will be integrated in the gitlab repos, so it can be used with CI/CD. Below are the cluster names for each environment:

1. Staging - ```staging-eks```
2. Prod    - ```prod-eks```
3. Dev     - ```dev-eks```

## Identity and Access Management

Three different aws accounts will be created to manage each environment respectively.

Each account has two different groups

- devops - Admin access given, as they are tasked with maintaining and debugging the state of the environment
- security - Read access given, as they need visibility into the platform and its operations

All the users in devops team should be added to the group devops and all the users in security should be added to security group 

## Deploying Website

Ghost is chosen for website creation, so a helm chart will be used to deploy ghost onto EKS cluster.

Installation will be automated to different environments using gitlab-ci pipeline in this repo.

There are six different jobs which can be used to install/uninstall Ghost on eks cluster.

## List of Jobs

All the jobs in the pipeline are manual and use both kubctl and helm to install and uninstall ghost on different eks clusters

1. Deploy Staging    - to install ghost to staging eks cluster
2. Un-Deploy Staging - to uninstall ghost on staging eks cluster
3. Deploy Prod       - to install ghost to prod eks cluster
4. Un-Deploy Prod    - to uninstall ghost on prod eks cluster
5. Deploy Dev        - to install ghost to dev eks cluster
6. Un-Deploy Dev     - to uninstall ghost on dev eks cluster

## Accessing Ghost

After successful installation of Ghost app on EKS cluster, following information can be used to access the application on internet:

1. Get the Ghost URL by running:
   
   - Blog URL  : http://ghost.staging.goprix.xyz/
   - Admin URL : http://ghost.staging.goprix.xyz/ghost
    
2. Get your Ghost login credentials by using:

    - Email: ```gopikrishna@goprix.xyz```
    - To extract password:
    ```shell
    $(kubectl get secret my-release-ghost -o jsonpath="{.data.ghost-password}" | base64 --decode)
    ```

## List of EKS Addons

After creating the EKS cluster, there are few addons which are installed for monitoring, visualising and debugging the EKS cluster and Ghost application in general for Devops and Security team

List of tools installed

- Prometheus
- Grafana
- Loki 
- Cert-manager
- Nginx Ingress Controller
- Atlantis
- Velero

All the add-ons are installed into namespace: ```gitlab-managed-apps```

A Gitlab pipeline is setup to automate their deployment into EKS cluster.

Refer to the repo [eks-addons](https://gitlab.com/nord-cloud-ghost/eks-addons) for more information.

## Disaster Recovery for EKS

Ghost will be deployed using AWS managed kubernetes cluster(EKS). 
Should there be a disaster, and the whole kubernetes cluster goes unavailable, 
then Velero deployed inside the EKS cluster will be used to recover all the workload upon recreating the EKS cluster.

Velero is used to take snapshots of the cluster workloads and stores it in both S3 and EBS snapshots.

Following steps can be followed for disaster recovery:
Note: Make sure to have access to EKS cluster and Velero installed on your local

1. To create periodical snapshots of the cluster, a schedule job can be created using velero. 
To achieve that, a simple EC2 instance which has access to EKS cluster and velero cli installed, then below command can be executed:

```shell 
velero schedule create <SCHEDULE NAME> --schedule "0 7 * * *"
```
This creates a Backup object with the name ```<SCHEDULE NAME>-<TIMESTAMP>```. The default backup retention period, expressed as TTL (time to live), is 30 days (720 hours); you can use the --ttl ```<DURATION>``` flag to change this as necessary

2. A disaster happens and you need to recreate your workloads on EKS cluster
3. After installing a new EKS cluster, deploy velero by configuring to same backup storage location, but this time with read-only access

```shell
kubectl patch backupstoragelocation <STORAGE LOCATION NAME> \
   --namespace velero \
   --type merge \
   --patch '{"spec":{"accessMode":"ReadOnly"}}'
```
4. Execute the command below to restore the snapshots

```shell
velero restore create --from-backup <SCHEDULE NAME>-<TIMESTAMP>
```

5. When ready, revert your backup storage location to read-write mode:

```shell
kubectl patch backupstoragelocation <STORAGE LOCATION NAME> \
   --namespace velero \
   --type merge \
   --patch '{"spec":{"accessMode":"ReadWrite"}}'
```

## Ghost flush (Serverless Function)
There is serverless function which allows you to delete all the posts in Ghost.

A lambda function is at your disposal which can be used to delete all the posts

## Ghost API

It’s possible to create and manage your content using the Ghost Admin API. 
Ghost Admin, uses the admin API - which means that everything Ghost Admin can do is also possible with the API, and a whole lot more!

## Creating a Session using Ghost API

curl -c ghost-cookie.txt -d username=<Username> -d password=<password> \
http://aaab6820f4fe340508fe7bf5cfa6a5de-1082779787.us-east-1.elb.amazonaws.com/ghost/api/canary/admin/session/

## Deleting all posts
curl -b ghost-cookie.txt \
-H "Content-Type: application/json" \
http://aaab6820f4fe340508fe7bf5cfa6a5de-1082779787.us-east-1.elb.amazonaws.com/ghost/api/canary/admin/db/
