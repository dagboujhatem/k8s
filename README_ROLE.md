# Kubernetes Roles docs


## Get all roles: 

    kubectl get clusterrole  --kubeconfig=./config.yml

You can pipe the result to filter it with `grep` command like: 

    kubectl get clusterrole  --kubeconfig=./config.yml | grep ingress-nginx

## Get all cluster role binding :

    kubectl get clusterrolebinding --kubeconfig=./config.yml

You can pipe the result to filter it with `grep` command like: 

    kubectl get clusterrolebinding --kubeconfig=./config.yml | grep ingress-nginx-admission

## Delete a role : 

    kubectl delete clusterrole ingress-nginx --kubeconfig=./config.
    
__NOTE:__ The `ingress-nginx` is the name of role. 

## Delete a cluster role binding : 

    kubectl delete clusterrole  ingress-nginx-admission --kubeconfig=./config.yml

__NOTE:__ The `ingress-nginx-admission` is the name of role binding. 