# Spring Boot-based Java web application

## Overview

This document outlines the setup and configuration of a Jenkins Pipeline for a Java-based application using a variety of DevOps tools and technologies, including Maven, SonarQube, Argo CD, Helm, and Kubernetes. This pipeline is designed to streamline the CI/CD process and ensure the efficient delivery of Java applications.

![flow](https://github.com/sagar-7227/spring-boot-app/assets/75033935/0c9d1803-3302-433d-8a57-019145daac32)

## Prerequisites

Before setting up the Jenkins Pipeline, ensure you have the following prerequisites in place:

- Jenkins CI/CD server installed and configured.
- Java development environment.
- Maven installed on the Jenkins server.
- SonarQube server set up and integrated with Jenkins.
- Argo CD installed and configured on your Kubernetes cluster.
- Helm installed on the Jenkins server.
- Access to a Kubernetes cluster for deployment.

### Install Jenkins.

Pre-Requisites:
 - Java (JDK)

### Run the below commands to install Java and Jenkins

Install Java

```
sudo apt update
sudo apt install openjdk-11-jre
```

Now, you can proceed with installing Jenkins

```
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
## Install the Docker Pipeline plugin in Jenkins:

   - Go to Manage Jenkins > Manage Plugins.
   - In the Available tab, search for "Docker Pipeline" and Install.

## Docker Slave Configuration


<img width="598" alt="Screenshot 2023-09-16 200706" src="https://github.com/sagar-7227/spring-boot-app/assets/75033935/c1ae48d7-f1a6-4c6a-9817-2d7a9a6d978a">


Run the below command to Install the Docker

```
sudo apt update
sudo apt install docker.io
```
 
### Grant Jenkins user and Ubuntu user permission to docker daemon.

```
sudo su - 
usermod -aG docker jenkins
usermod -aG docker ubuntu
systemctl restart docker
```

Once you are done with the above steps, it is better to restart Jenkins.


<img width="698" alt="Screenshot 2023-09-16 200906" src="https://github.com/sagar-7227/spring-boot-app/assets/75033935/bdf81341-6426-4e29-bdea-5932c083e46e">


```
http://<azure-instance-public-ip>:8080/restart
```

### Configure a Sonar Server locally


<img width="765" alt="Screenshot 2023-09-16 200943" src="https://github.com/sagar-7227/spring-boot-app/assets/75033935/2d86a031-545e-441d-a71c-611f3b8efc26">


```
apt install unzip
adduser sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip *
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start
```

Now you can access the `SonarQube Server` on `http://<ip-address>:9000`

### Accessing Minikube Cluster locally


<img width="825" alt="Screenshot 2023-09-16 211310" src="https://github.com/sagar-7227/spring-boot-app/assets/75033935/1bf147b7-0b30-4430-bfed-6e49e02fdd82">


```
sagar@SAGAR-REALME MINGW64 ~/OneDrive/Desktop$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```
## Creating ArgoCd Deployment

```
sagar@SAGAR-REALME MINGW64 ~/OneDrive/Desktop
$ cat argocd-basic.yml
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: example-argocd
  labels:
    example: basic
spec: {}
```

```
sagar@SAGAR-REALME MINGW64 ~/OneDrive/Desktop
$ kubectl apply -f argocd-basic.yml
argocd.argoproj.io/example-argocd created

```
## Accessing Argocd pods and Service

```
sagar@SAGAR-REALME MINGW64 ~/OneDrive/Desktop
$ kubectl get pods
NAME                                          READY   STATUS    RESTARTS      AGE
example-argocd-application-controller-0       1/1     Running   1 (24m ago)   29m
example-argocd-redis-6b8667cdb8-286fl         1/1     Running   1 (24m ago)   29m
example-argocd-repo-server-5d547c6f69-qt68t   1/1     Running   1 (24m ago)   29m
example-argocd-server-bbdf5fdff-d8klb         1/1     Running   1 (24m ago)   29m

```

```
sagar@SAGAR-REALME MINGW64 ~/OneDrive/Desktop
$ kubectl get svc
NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
example-argocd-metrics          ClusterIP   10.105.42.39     <none>        8082/TCP                     34m
example-argocd-redis            ClusterIP   10.96.7.202      <none>        6379/TCP                     34m
example-argocd-repo-server      ClusterIP   10.109.131.50    <none>        8081/TCP,8084/TCP            34m
example-argocd-server           NodePort    10.105.76.92     <none>        80:31859/TCP,443:31964/TCP   34m
example-argocd-server-metrics   ClusterIP   10.105.158.194   <none>        8083/TCP                     34m
kubernetes                      ClusterIP   10.96.0.1        <none>        443/TCP                      135m

```
## Start Argocd Service and Access it

```
sagar@SAGAR-REALME MINGW64 ~/OneDrive/Desktop
$ minikube service  example-argocd-server
|-----------|-----------------------|-------------|---------------------------|
| NAMESPACE |         NAME          | TARGET PORT |            URL            |
|-----------|-----------------------|-------------|---------------------------|
| default   | example-argocd-server | http/80     | http://192.168.49.2:31859 |
|           |                       | https/443   | http://192.168.49.2:31964 |
|-----------|-----------------------|-------------|---------------------------|
* Starting tunnel for service example-argocd-server.
|-----------|-----------------------|-------------|------------------------|
| NAMESPACE |         NAME          | TARGET PORT |          URL           |
|-----------|-----------------------|-------------|------------------------|
| default   | example-argocd-server |             | http://127.0.0.1:65124 |
|           |                       |             | http://127.0.0.1:65125 |
|-----------|-----------------------|-------------|------------------------|
[default example-argocd-server  http://127.0.0.1:65124
http://127.0.0.1:65125]
! Because you are using a Docker driver on windows, the terminal needs to be open to run it.

```

<img width="1065" alt="Screenshot 2023-09-16 205114" src="https://github.com/sagar-7227/spring-boot-app/assets/75033935/e9de3660-bec7-4838-8bb9-87d2b0c0ac8a">

<div align="center">
<i>Other places you can find me:</i><br> 
<br>
<a href="https://www.linkedin.com/in/sagar-vashnav/" target="_blank"><img src="https://img.shields.io/badge/linkedin-%230077B5.svg?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn"></a>
</div>
