# Jenkins in Kubernetes

Configuration of Jenkins in Kubernetes to run distributed builds.


## Prerequisites

* Linux


## Setup

We will here install Kubernetes with Minikube, for this, you need to install Minikube.
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/
```
  
When the installation is complete, you can now start minikube who will install kubernetes.
```
minikube start
```
  
You can after that check that minikube is running.
```
minikube status
```
  
We also will need kubectl.
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```
  
We will now build our docker image for Jenkins, you need to be in this github repository where is located the Dockerfile.
```
eval $(minikube docker-env)
docker build -t jenkins-image:1.0 .
```
  
You can check your new image.
```
docker images
```

Now create a namespace.
```
kubectl create namespace jenkins
```

To authorise Jenkins to access Kubernetes, you need to create a serviceAccount.
```
kubectl create serviceaccount jenkins -n jenkins
```
  
We now going to apply all our yaml files.
```
kubectl apply -f jenkins-deployment.yaml -n jenkins
kubectl create -f jenkins-service.yaml -n jenkins
kubectl apply -f jenkins-serviceAccount.yaml -n jenkins
```

We can check all that in the minikube dashboard.
```
minikube dashboard
```

For the configuration of Jenkins, we will need the IP address of kubernetes and jenkins.
```
kubectl cluster-info
```

The port of jenkins is show with this command.
```
kubectl get service --namespace jenkins
```

You will also need the secret and certificate of the service account, copy output of these commands.
```
kubectl get secret $(kubectl get sa jenkins -n jenkins -o jsonpath={.secrets[0].name}) -n jenkins -o jsonpath={.data.token} | base64 --decode
kubectl get secret $(kubectl get sa jenkins -n jenkins -o jsonpath={.secrets[0].name}) -n jenkins -o jsonpath={.data.'ca\.crt'} | base64 --decode
```

Go now to Jenkins and use the password you will get in the logs of the created pod.
```
kubectl get pods -n jenkins
kubectl logs {pod NAME} -n jenkins
```

You have now access to Jenkins, first you have to create a Secret Text credential with the ouput of the first command `kubectl get secret`.

We have now to configure Kubernetes plugin in Jenkins, go to Cloud Configuration and put in :
* Kubernetes URL
* Kubernetes server certificate key
* Kubernetes namespace
* Jenkins credentiales you just created
* Jenkins URL

Add a pod template and a container template, add in :
* Docker image 'jenkins/inbound-agent:4.3-4'

You are now ready to check your configuration by run many parallel jobs in Jenkins !
