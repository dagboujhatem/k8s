# Kubernetes docs

In this docs, we present the most important commands to learn k8s. All these commands are tested in a k8s cluster hosted at [OVH Public Cloud](https://www.ovhcloud.com/fr/public-cloud/?at_medium=sl&at_platform=google&at_campaign=AdWords&at_creation=noi_ovh_fr_se_enterprise_publiccloud_defensive(Public%20Cloud%20-%20Generic)&at_variant=676696754590&at_network=search&at_term=public%20cloud%20ovh&gad_source=1&gclid=EAIaIQobChMIx_Kwn9SogwMVEkFBAh1EMgJNEAAYASAAEgJgufD_BwE).

## 1. Install kubectl guide:

Please check [this link](https://kubernetes.io/docs/tasks/tools/) to install kubectl.

## 2. Get cluster infos:

In this section we present the basic commands for managing our k8s cluster.

- Get kubectl version:

        kubectl version --client

- Get cluster info:

        kubectl cluster-info --kubeconfig=./config.yml

- Get cluster nodes:

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

        kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret --kubeconfig=./config.yml | grep admin-user-token | awk '{print $1}') --kubeconfig=./config.yml

### Access the Dashboard: 
 
        kubectl proxy --kubeconfig=./config.yml
  
### To delete all kubernetes-dashboard resources: 

To remove all resources created by your previous kubernetes-dashboard deployment, just execute the following command line:

        kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml --kubeconfig=./config.yml
To romove the Role Binding, just execute the following command line:

        kubectl delete -f dashboard-cluster-role-binding.yml --kubeconfig=./config.yml

## 4. Deploying an application with k8s:

Based on [this tutorial](https://help.ovhcloud.com/csm/en-gb-public-cloud-kubernetes-deploy-application?id=kb_article_view&sysparm_article=KB0049713), we can deploy our first application in Kubernetes.

### Step 0 - Create your first namespace 

        kubectl create ns hello-app --kubeconfig=./config.yml

### Step 1 - Deploy your first application 

        kubectl apply -f hello.yml -n hello-app --kubeconfig=./config.yml

### Step 2 - List the pods

        kubectl get pods -n hello-app -l app=hello-world --kubeconfig=./config.yml

### Step 3 - List the deployments

        kubectl get deploy -n hello-app -l app=hello-world --kubeconfig=./config.yml

### Step 4 - List the services

        kubectl get services -n hello-app -l app=hello-world --kubeconfig=./config.yml

### Step 5 - List the services

        kubectl describe service --kubeconfig=./config.yml

### Step 6 - Test your service

    export SERVICE_URL=$(kubectl get svc -n hello-app -l app=hello-world --kubeconfig=./config.yml -o jsonpath='{.status.loadBalancer.ingress[].ip}')

__Notes:__ If this way not work for you, please go to load balancer in OVH dashboard to get the ip adress.

### Step 7 - Clean up

    kubectl delete ns hello-app --kubeconfig=./config.yml

