# Kubernetes docs

In this docs, we present the most important commands to learn k8s. All these commands are tested in a k8s cluster hosted at [OVH Public Cloud](https://www.ovhcloud.com/fr/public-cloud/?at_medium=sl&at_platform=google&at_campaign=AdWords&at_creation=noi_ovh_fr_se_enterprise_publiccloud_defensive(Public%20Cloud%20-%20Generic)&at_variant=676696754590&at_network=search&at_term=public%20cloud%20ovh&gad_source=1&gclid=EAIaIQobChMIx_Kwn9SogwMVEkFBAh1EMgJNEAAYASAAEgJgufD_BwE).

## 1. Install kubectl guide:

Please check [this link](https://kubernetes.io/docs/tasks/tools/) to install kubectl.

## 2. Get cluster infos:

In this section we present the basic commands for managing our k8s cluster.

- Get kubectl version:

        kubectl version --client

- Get cluster infos (like: CoreDNS, proxy):

        kubectl cluster-info --kubeconfig=./config.yml

- Get clusters infos (like: Name, Type, IP, Port):

        kubectl get svc --kubeconfig=./config.yml
- Get cluster details (like: Name, Type, IP, Port):

        kubectl describe svc CLUSTER_NAME --kubeconfig=./config.yml

- Get all namespaces in the cluster k8s:

        kubectl get namespaces --kubeconfig=./config.yml

- Get all cluster nodes:

        kubectl get nodes --kubeconfig=./config.yml

- Describe cluster nodes:

        kubectl describe service --kubeconfig=./config.yml

- Get cluster users:

        kubectl config get-users --kubeconfig=./config.yml

- Get cluster pods:

        kubectl get pods --kubeconfig=./config.yml

 ## 3. Install k8s dashboard: 

Based on [this tutorial](https://help.ovhcloud.com/csm/en-gb-public-cloud-kubernetes-install-kubernetes-dashboard?id=kb_article_view&sysparm_article=KB0049828), we can install the Kubernetes Dashboard.
 
 ### Deploy the Dashboard in your cluster: 

        kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml --kubeconfig=./config.yml

### Create An Authentication Token (RBAC)

- Step 1 - Create Service Account:

        kubectl apply -f dashboard-service-account.yml --kubeconfig=./config.yml

- Step 2 - Create a RoleBinding: 

        kubectl apply -f dashboard-cluster-role-binding.yml --kubeconfig=./config.yml

- Step 3 - Create a Secret ServiceAccountToken: 

        kubectl apply -f service-account-token.yml --kubeconfig=./config.yml

- Step 4 - Get Bearer Token: 

Now we need to find token we can use to log in. Execute following command:

For Bash:

        kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret --kubeconfig=./config.yml | grep admin-user-token | awk '{print $1}') --kubeconfig=./config.yml

For Powershell (To verfiy):

        kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | sls admin-user --kubeconfig=./config.yml | ForEach-Object { $_ -Split '\s+' } | Select -First 1) --kubeconfig=./config.yml

### Access the Dashboard: 
 
        kubectl proxy --kubeconfig=./config.yml
  
### To delete all kubernetes-dashboard resources: 

To remove all resources created by your previous kubernetes-dashboard deployment, just execute the following command line:

        kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml --kubeconfig=./config.yml
To romove the Role Binding, just execute the following command line:

        kubectl delete -f dashboard-cluster-role-binding.yml --kubeconfig=./config.yml

## 4. Deploying an application with k8s:

Based on [this tutorial](https://help.ovhcloud.com/csm/en-gb-public-cloud-kubernetes-deploy-application?id=kb_article_view&sysparm_article=KB0049713), we can deploy our first application in Kubernetes.

### Step 0 - Create your first namespace:

        kubectl create ns hello-app --kubeconfig=./config.yml

### Step 1 - Deploy your first application:

        kubectl apply -f hello.yml -n hello-app --kubeconfig=./config.yml

### Step 2 - List the pods:

        kubectl get pods -n hello-app -l app=hello-world --kubeconfig=./config.yml

### Step 3 - List the deployments:

        kubectl get deploy -n hello-app -l app=hello-world --kubeconfig=./config.yml

### Step 4 - List the services:

        kubectl get services -n hello-app -l app=hello-world --kubeconfig=./config.yml

### Step 5 - Describe all services:

        kubectl describe service --kubeconfig=./config.yml

### Step 6 - Test your service:

    export SERVICE_URL=$(kubectl get svc -n hello-app -l app=hello-world --kubeconfig=./config.yml -o jsonpath='{.status.loadBalancer.ingress[].ip}')

__Notes:__ If this way not work for you, please go to load balancer in OVH dashboard to get the IP address.

### Step 7 - Clean up:

    kubectl delete ns hello-app --kubeconfig=./config.yml



## 5. Install & deploy Jenkins with k8s:

Based on [this tutorial](https://help.ovhcloud.com/csm/en-public-cloud-kubernetes-install-jenkins?id=kb_article_view&sysparm_article=KB0049821), we can install & deploy Jenkins in Kubernetes.

###  Before you begin: 

This tutorial presupposes that you already have a working OVHcloud Managed Kubernetes cluster.

You also need to have Helm installer on your workstation and your cluster,  please refer to [this link](https://github.com/helm/helm/releases) to install Helm and you don't forget to add it to environnement variables.

### Step 1 - Installing the Jenkins Helm chart: 
    helm install jenkins bitnami/jenkins --version 12.4.8 --kubeconfig=./config.yml

This will install your Jenkins master.

Use the following command if you want verify the list of installation: 

        helm repo list

### Step 2 - Get the LoadBalancer URL: 

As the instructions say, you will need to wait a few moments to get the LoadBalancer URL. You can test if the LoadBalancer is ready using:

    kubectl get svc --namespace default -w jenkins --kubeconfig=./config.yml

The URL under __EXTERNAL-IP__ is your Jenkins URL. You can the follow the instructions on the Helm Chart to get the connection parameters.

### Step 3 - Get the connection parameters: 

Use the following command to get the password from the secret jenkins: 

    kubectl get secret --namespace default jenkins -o jsonpath="{.data.jenkins-password}" --kubeconfig=./config.yml | base64 --decode 

You can also, use the __-d__ flag to decode the value of secret, like the following command: 

    kubectl get secret --namespace default jenkins -o jsonpath="{.data.jenkins-password}" --kubeconfig=./config.yml | base64 -d

__Note:__ The username is __user__.


### Step 4 - Cleaning up:

To clean up your cluster, simply use Helm to delete your Jenkins release.

    helm delete jenkins --kubeconfig=./config.yml


## 6. Install & deploy the Ingress Controller with OVH Cloud:

Based on [this tutorial](https://help.ovhcloud.com/csm/fr-public-cloud-kubernetes-install-nginx-ingress?id=kb_article_view&sysparm_article=KB0055171), we can install & deploy the ingress controller with OVH Cloud.

You can learn more in [this Helm](https://kubernetes.github.io/ingress-nginx/deploy/#ovhcloud) package.

###  Before you begin: 

This tutorial presupposes that you already have a working OVHcloud Managed Kubernetes cluster.

You also need to have Helm installer on your workstation and your cluster,  please refer to [this link](https://github.com/helm/helm/releases) to install Helm and you don't forget to add it to environnement variables.

### Let's define Ingress Resource: 

In Kubernetes, an Ingress resource allows you to access to your Services from outside the cluster. The goal is to avoid creating a Load Balancer per Service but only one per Ingress.

![Nginx Ingress Controller](https://help.ovhcloud.com/public_cloud-containers_orchestration-managed_kubernetes-installing-nginx-ingress-images-ingress.png)


### Step 1 - Install Ingress Nginx Ingress Controller with Helm:

- __First way : using Helm:__

If you have Helm installed, you can run the folloming command to install it: 

        helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

Use the following command if you want verify the list of installation: 

        helm repo list

Use the following command if you want update all Helm package: 

        helm repo update

Use the following command if you want install the Helm package:

        helm -n ingress-nginx install ingress-nginx ingress-nginx/ingress-nginx --create-namespace  --kubeconfig=./config.yml

- __Second way : using kubectl:__

If you don't have Helm or if you prefer to use a YAML manifest, you can run the following command instead:

       kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml --kubeconfig=./config.yml

### Step 2 - Online testing:

As the instructions say, you will need to wait a few moments to get the LoadBalancer URL. You can test if the LoadBalancer is ready using:

    kubectl get service ingress-nginx-controller --namespace=ingress-nginx --kubeconfig=./config.yml

The URL under __EXTERNAL-IP__ is your nginx-ingress URL. You can then access your nginx-ingress at http://[__EXTERNAL-IP__] via HTTP or https://[__EXTERNAL-IP__] via HTTPS.

### Step 3 - Alternative to get nginx-ingress URL :

Enter the following command to retrieve it:

        kubectl get svc ingress-nginx-controller -n ingress-nginx -o jsonpath='{.status.loadBalancer.ingress[].ip}' --kubeconfig=./config.yml

### Step 4 - Cleaning up:

- __First way : using Helm:__
       
       helm delete ingress-nginx --kubeconfig=./config.yml

- __Second way : using kubectl:__

       kubectl delete ns ingress-nginx  --kubeconfig=./config.yml