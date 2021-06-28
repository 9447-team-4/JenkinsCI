## Installing Jenkins on kubernetes

# Create namespace
  - kubectl create namespace jenkins

# Define Deployment
  - kubectl create -f jenkins.yaml --namespace jenkins

# Expose the pod created above using NodePort service
  - kubectl create -f jenkins-service.yaml --namespace jenkins

## Accessing the Jenkins UI

# Retrieve your node IP
  - kubectl get nodes -o wide

The output will have various nodes with external IPs.Copy one of the external_ip values (for minikube, copy the internal_ip) an access the UI at
  - http://ip:30000

Page will appear asking for administrator password and instructions on retrieving the password from the pod logs.

## Getting password from kubectl
  - kubectl get pods -n jenkins

Get the default password from withing the log name of the pod.
  - kubectl logs ${pod_name} -n jenkins

## Gitea Integration

For Gitea integration, we will not be using gitea plugin but creating a webhook in gitea and linking it with Jenkins Project.

### Jenkins Setup

Create a new Project in Jenkins. This project will be using git for SCM (Source Code Management).

1) Install the Gogs plugin.
2) Click on 'New Item' option on the Jenkins UI Dashboard.
3) Enter the name of the Project and select 'Freestyle Project' option. Click OK.
4) In the 'Source Code Management' section, select git.
5) Specify the repository url
  - For minikube -> http://gitea-charts-http.default.svc.cluster.local:3000/${repoName}.git
6) Add credentials 
   - Click Add -> Jenkins.
   - For Kind as 'Username with password'
     -- Enter the username of Gitea Account you want to use.
     -- Enter the password for the account.
     -- Give it an ID.
     -- Click Add. You should now be able to Select the created Credential from the drop down.
     -- Goto step 7.
   - For Kind as 'Gitea Personal Access Token'
     -- Go to gitea and login with the account you want to use for Jenkins Credentials.
     -- Go to account settings and click on 'Applications' tab.
     -- Give the token a name and click 'Generate Token'.
     -- Copy this token and paste it in the 'Token' field in Jenkins Add Credential setting.
     -- Give this token an ID and click add. You should now be able to Select the created Credential from the drop down.
7) In the 'Build Triggers' section, select 'Poll SCM' and leave the Schedule blank. Leaving it blank tells Jenkins to only run when triggered by a hook.
8) Change other settings for the build as you wish and click Save.

### Gitea Setup

1) Go to the webhook setting for the repo you want.
2) Click 'Add webhook' and select the type Gogs.
3) In the url type - http://192.168.99.122:30000/gogs-webhook/?job=${jenkins_project_name}
4) Make sure Trigger On 'Push Event' is selected.
5) Click Update Webhook.
6) Test the webhook by clicking 'Test Delivery'. This should trigger the project in Jenkins.
7) The Jenkins dashboard will show your current project with the 'Last Success' field.



