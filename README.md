# JenkinsCI

## Create Namespace for Jenkins
  - kubectl create namespace jenkins

## Install Helm CLI
  - curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
  - chmod 700 get_helm.sh
  - ./get_helm.sh

## Configure Helm
  - helm repo add jenkinsci https://charts.jenkins.io
  - helm repo update
  - The helm charts in the Jenkins repo can be listed with the command:
    -> helm search repo jenkinsci

## Create a Persistent Volume
This will prevent us from losing our whole configuration of the Jenkins controller and our jobs when we reboot our minikube. In a multi-node Kubernetes cluster, you’ll need some solution like NFS to make the mount directory available in the whole cluster. But because we use minikube which is a one-node cluster we don’t have to bother about it. 

We choose to use the /data directory. This directory will contain our Jenkins controller configuration.
  - kubectl apply -f jenkins-volume.yaml

## Create a service account
  - kubectl apply -f jenkins-sa.yaml

## Install Jenkins
  - chart=jenkinsci/jenkins
  - helm install jenkins -n jenkins -f jenkins-values.yaml $chart

## Get admin password
  - jsonpath="{.data.jenkins-admin-password}"
  - secret=$(kubectl get secret -n jenkins jenkins -o jsonpath=$jsonpath)
  - echo $(echo $secret | base64 --decode)

## Get the Jenkins URL to visit by running these commands in the same shell:
  - jsonpath="{.spec.ports[0].nodePort}"
  - NODE_PORT=$(kubectl get -n jenkins -o jsonpath=$jsonpath services jenkins)
  - jsonpath="{.items[0].status.addresses[0].address}"
  - NODE_IP=$(kubectl get nodes -n jenkins -o jsonpath=$jsonpath)
  - echo http://$NODE_IP:$NODE_PORT/login



