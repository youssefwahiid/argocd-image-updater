 ArgoCD Image Updater Setup and Workflow
1. Project Overview
   
    This project demonstrates how to automate Kubernetes application deployments using ArgoCD and ArgoCD Image Updater.
    Whenever a new Docker image version is pushed to Docker Hub, ArgoCD Image Updater automatically updates the image tag in the GitHub manifests and synchronizes the               deployment — achieving a fully automated CI/CD flow.
------------------------------------------------------------------------------------------------------------------------------

2. Setup Steps
    a. ArgoCD Installation
        Installed ArgoCD on the Kubernetes cluster. 
        Verified that the ArgoCD web UI and CLI were working correctly.
  
    b. Application Manifests 
        Created the following manifest files in VS Code:
        deployment.yaml
        service.yaml
        kustomization.yaml
   
    Added all files to a GitHub repository.
------------------------------------------------------------------------------------------------------------------------------

3. GitHub Configuration

    Created a Fine-grained Personal Access Token (PAT) with the following permissions:
    
    Administration
    Commit statuses
    Contents
    Deployments
    Metadata
   
    This PAT allows ArgoCD Image Updater to commit updates (new image tags) directly to the GitHub repo.

------------------------------------------------------------------------------------------------------------------------------

4. Git Credentials Secret
   
    Created a Kubernetes secret to store GitHub credentials securely:
    kubectl -n argocd create secret generic git-creds \
      --from-literal=username=youssefwahiid \
      --from-literal=password=<your_github_pat>

------------------------------------------------------------------------------------------------------------------------------

5. ArgoCD Application Setup

    Added the application to the ArgoCD dashboard.
    Installed ArgoCD Image Updater.
    Edited the application configuration using:
    ---> kubectl edit app -n argocd webapp
    
    Added the following annotations under metadata to enable automatic image updates:
    
    annotations:
   
      argocd-image-updater.argoproj.io/image-list: youssefwaheeds/nodeapp:v1.x
   
      argocd-image-updater.argoproj.io/nodeapp.update-strategy: semver
   
      argocd-image-updater.argoproj.io/write-back-method: git:secret:argocd/git-creds
------------------------------------------------------------------------------------------------------------------------------
6. Image Update Process
    Built and tagged a new Docker image:
    
    docker tag nodeapp:v1.1 youssefwaheeds/nodeapp:v1.7
    docker push youssefwaheeds/nodeapp:v1.7
    
    ArgoCD Image Updater automatically detected the new tag.
    It updated the GitHub manifest (.argocd-source-webapp.yaml) with the new version.
    ArgoCD then synchronized the application with the new image automatically.

------------------------------------------------------------------------------------------------------------------------------
7. Verification

    Checked logs to confirm the update process:
   
    kubectl logs -n argocd <image-updater-pod-name>
    
    Verified that GitHub was updated automatically and the cluster was running the latest image.
