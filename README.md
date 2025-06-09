# How to make and deploy a HELM Chart
## Contents
1. What is HELM?
2. What is a HELM chart?
3. How do I make a HELM chart?

## What is HELM?
HELM is a very useful tool for Kubernetes development and deployment.  K8s doesn't recongize your application as a unit -- it just takes the YAML you feed it and makes it happen on the cluster.  HELM sees your packages wholistically and as belonging together and depending on one another.  Hence it's name: a package manager for Kubernetes.

This guide assumes you already have a Kubernetes application in mind that you want to convert into a HELM chart.  We'll be using <a href="https://github.com/khutchi2/text-db-webapp"> text-db-webapp </a>, but the process here should generalize to really any K8s application.

This guide walks through taking the text-db-webapp and turning it into a HELM chart.

## Steps
### 0. Prerequisites and Dependencies
1. Install HELM and Kubectl
- https://helm.sh/docs/intro/install/ 
- https://kubernetes.io/docs/tasks/tools/ 
2. Clone the <a href="https://github.com/khutchi2/text-db-webapp"> text-db-webapp </a> repo
### 1. Create your HELM repo
The command for creating a HELM repo is:
```bash
helm create <name-of-repo>
```
We're going to be calling the repo "text-db-webapp" so specifically we'll run
```bash
helm create text-db-webapp
```
### 2. Delete everything in `/templates/.` and `values.yaml`
```bash
rm -rf /templates/*
```
### 3. Copy in everything from <a href="https://github.com/khutchi2/text-db-webapp/tree/main/k8s/individual_manifests">`` </a>
- If you cloned the text-db-webapp GitHub repo you can just copy from the folder in your local copy of the repo.  If you didn't clone it, from the link above, download all of the manifests and copy them into `/templates`
- The `/templates` folder should contain all of your various cluster objects like deployments, services, ingresses, etc.
### 4. Update `values.yaml`
- The `values.yaml` file provides a central spot for all of your...values.  HELM has an "autocompletion" feature and the `values.yaml` file is a specific dictionary for that
- The amount of this you leverage is up to your judgement.  For something smaller, it may not even be necessary at all.  If you were to put every value in `values.yaml` you might find yourself having a hard time tracking what the values are in a particular manifest because you'll be constantly looking back and forth
- The `values.yaml` file will be the basic YAML `key:value` structure:
```yaml
appName: flask-app
```
and in the manifest you can reference that as follows:
```yaml
  template:
    metadata:
      labels:
        app: {{.Values.appName}}
```
### 5. Installing the application
- Once you have everything setup to your liking, installing the app onto your cluster is as simple as running the following:
```bash
helm install <release-name> <helm-repo-location>
```
### 6. Adding release notes
- Create `/templates/NOTES.txt`
- Anything you write in here will be displayed after installing or upgrading.  For example:
```txt
To view the UI from a web browser run:

kubectl port-forward svc/nginx-service 8080:80
```
### 7. Upgrading the application (Optional)
- When you inevitably make an update or change, instead of having to reinstall the whole application, you can use the `helm upgrade` command.  For example, say you made some changes to the `values.yaml` file.  To apply those changes, you can run:
```bash
helm upgrade <release-name> <helm-repo-location> --values <path_to_values.yaml>
```