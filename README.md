# helm-security-demo
Helm chart that demonstrates how to do project management


Securing Kubernetes Helm

My cluster is k8s.mmerrilldev.com

1) update your kops context to use an oidc provider.  In my case, google.
2) Under the kubeAPIServer section in your kops cluster config, add oidc info

    oidcClientID: 9**********-********************.apps.googleusercontent.com
    oidcIssuerURL: https://accounts.google.com
    oidcUsernameClaim: email
    
3) kops update cluster --yes
4) login and get a token.  We use k8s-auth. (https://confluence.vonage.com/display/BackEndDevelopmentTeam/How+to+use+kubernetes+sandbox)
5) Set your context:

**kubectl config set-context $(kubectl config current-context) --user=jjpaacks@gmail.com**

7) You will see permissions issues, the user doesn't exist in RBAC yet
8) To go back to admin, run 

**kubectl config set-context $(kubectl config current-context) --user=k8s.mmerrilldev.com**

10) To generate a CA, look here: https://github.com/helm/helm/blob/master/docs/tiller_ssl.md
11) As the admin, we need to create the helm for our admin charts, and be able to install the project chart:
12) Install service account and cluster role binding for RBAC:
13) apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller-deploy
  namespace: kube-system
  apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller-deploy
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller-deploy
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: tiller-deploy
  namespace: kube-system
  
  
13) helm init --debug --service-account tiller-deploy --tiller-tls --tiller-tls-cert ./demo-tiller.cert.pem --tiller-tls-key ./demo-tiller.key.pem --tiller-tls-verify --tls-ca-cert demo-ca.cert.pem --history-max 10
14) I like to link the certs in my .helm directory, so helm picks them up
    
    **ln -s -f /Users/mmerrill/.helm/mmerrill-helm-dev.cert.pem /Users/mmerrill/.helm/cert.pem
    ln -s -f /Users/mmerrill/.helm/mmerrill-helm-dev.key.pem /Users/mmerrill/.helm/key.pem
    ln -s -f /Users/mmerrill/.helm/ca-dev.cert.pem /Users/mmerrill/.helm/ca.pem**
    
Create our namespaces we'll be using, this can be automated, just haven't gotten to it yet:

**kubectl create namespace coreservices-tiller
kubectl create namespace security
kubectl create namespace dev-test
kubectl create namespace someone-else-ns
kubectl create namespace someone-else-tiller**



    
Install the project helm chart:

helm install . --name rbac-local -f k8s-mmerrill.yaml --tls --debug --dry-run


Here is the helm chart  https://github.com/mmerrill3/helm-security-demo



To test it out:
kubectl config set-context $(kubectl config current-context) --user=jjpaacks@gmail.com
kubectl run -ti --rm --image busybox busybox -- sh





    
