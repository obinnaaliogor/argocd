argo cd:

# install ArgoCD in k8s
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# access ArgoCD UI
kubectl get svc -n argocd
kubectl port-forward svc/argocd-server 8080:443 -n argocd

# login with admin user and below token (as in documentation):
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo

# you can change and delete init password

# Switch to argodc namespace
# Modify the application.yaml with your git repository and apply to the cluster.
kubectl apply -f application.yaml

# Apply the secret in your cluster. In place of password, use token
kubectl apply -f secrets.yaml

# Push your manifest files to your repo


a) Push deployment and service to your repository
deployment.yaml
================
apiVersion: apps/v1
kind: Deployment
metadata:
name: myapp
spec:
selector:
matchLabels:
app: myapp
replicas: 2
template:
metadata:
labels:
app: myapp
spec:
containers:
- name: myapp
image: obinnaaliogor/hello-world
ports:
- containerPort: 8080

service.yaml
=============
apiVersion: v1
kind: Service
metadata:
name: myapp-service
spec:
selector:
app: myapp
ports:
- port: 8080
  protocol: TCP
  targetPort: 8080

b) Apply the application.yaml and secret.yaml to the cluster

application.yaml
================
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
name: myapp-argo-application
namespace: argocd
spec:
project: default

source:
repoURL: https://github.com/yourrepo.git
targetRevision: HEAD
path: dev
destination:
server: https://kubernetes.default.svc
namespace: myapp

syncPolicy:
syncOptions:
- CreateNamespace=true

    automated:
      selfHeal: true
      prune: true

secret.yaml
============
NOTE IF YOUR REPOSITORY IS NOT A PUBLIC REPO, THEN YOU HAVE TO AUTHORIZE ARGOCD
TO DEPLOY THE APPLICATION USING THE FOLLOWING MANIFEST FILE DEPLOYED IN YOUR K8S CLUSTER.

apiVersion: v1

kind: Secret

metadata:
name: argocd-repo

namespace: argocd

labels:
argocd.argoproj.io/secret-type: repository #targeting repository..

stringData:

type: git

url: https://github.com/yourrepo.git  #repo url of your github

password:  your-token
  
username: yourusername

ONE AMONGST THE GOOD THING ARGOCD PROVIDES IS THAT THE APPLICATION RESOURCE GIVES YOU THE PRIVILEGE TO SET THE HEAD WHICH THE MAIN BRANCH FOR DEPLOYMENT.
WHICH MEANS YOU RUN YOUR CODE TEST USING THE SHIFT LEFT POLICY AND ALSO THE USE OF GITHUB ACTIONS USING SONARCLOUD/SONARQUBE..
ONLY ON PULL REQUEST AND MERGE TO THE MAIN BRANCH WILL ARGOCD EFFECT ANY DEPLOYMENT TO YOUR CLUSTER...
ALSO NOTE THAT THE REPO IS THE ONLY SOURCE OF TRUTH AND IF ANYTHING ON THE CHANGES IN YOUR CLUSTERS, ARGOCD WILL REVERT THE CHANGES.
