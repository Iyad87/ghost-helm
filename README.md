# Ghost-helm

Ghost installation is automated to different environments using gitlab-ci pipeline in this repo.

There are six different jobs which can be used to install/uninstall Ghost on eks cluster.

## List of Jobs

All the jobs in the pipeline are manual and use both kubctl and helm to install and uninstall ghost on different eks clusters

1. Deploy Staging    - to install ghost to staging eks cluster
2. Un-Deploy Staging - to uninstall ghost on staging eks cluster
3. Deploy Prod       - to install ghost to prod eks cluster
4. Un-Deploy Prod    - to uninstall ghost on prod eks cluster
5. Deploy Dev        - to install ghost to dev eks cluster
6. Un-Deploy Dev     - to uninstall ghost on dev eks cluster

## List of EKS cluster

Three EKS clusters are integrated in the current gitlab project. Each EKS cluster refers to each environment

1. Staging - ```staging-eks```
2. Prod    - ```prod-eks```
3. Dev     - ```dev-eks```
