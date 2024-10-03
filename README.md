# Canary Deployment for Safer Kubernetes Updates

# Problem Statement:

An organization running a cloud-native application on Kubernetes is looking to improve its deployment process to reduce the risk of downtime and application errors during updates. The current approach uses full, immediate rollouts, which can lead to issues like service outages, bugs in the new version impacting all users, and a lack of quick rollback options. The company needs a more reliable deployment strategy that allows them to gradually introduce changes, monitor the new version's performance, and minimize the impact of potential issues on end-users.

The organization seeks to implement a Canary Deployment strategy on Kubernetes to incrementally release new application versions to a small subset of users. This approach will allow the team to gather feedback on the new version's stability and performance while ensuring the majority of users continue to interact with the stable version. The goal is to significantly reduce downtime and improve the ability to quickly roll back or fix issues if they arise during deployment.

This project aims to design and deploy a Canary Deployment pipeline within the organization’s Kubernetes cluster, leveraging advanced traffic routing to ensure a smooth and controlled application update process.


## So what is Canary Deployment? 
A canary deployment strategy is a strategy that rolls out a new version of your application to a small group of users, while the majority of users continue the old version of the application. If the new version works as expected, it gradually replaces the old version.


# Steps for Implementing Canary Deployment

# 1. Set Up a Kubernetes Cluster
You can set up your cluster using the following;
- Minikube
- Docker Desktop
- Kind (Kubernetes IN Docker)
- EKS

