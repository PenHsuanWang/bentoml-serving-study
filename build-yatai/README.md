# Using Yatai to Deploy Bento on K8S

:::info
:warning:  **Yatai Deployment** using `HorizontalPodAutoscaler` Api Version `autoscaling/v2beta2` which is **deprecated** in kubernetes `v1.23+` and unavailable in `v1.26+`.
https://github.com/kubernetes/kube-state-metrics/issues/1711

:bug: Using kubernetes `v1.22` to do the following job will be fine.
:::

## Introduction to Yatai(やたい)
The Yatai(やたい->屋台) is the food cart in Japaness word. Using to serve bento(弁当) to customer.
In bentoml, we build our model as `bento` in development environment. Once the Model is quilified to go production. Yatai can help to deploy `bento` to K8S.

## Architecture

![](https://i.imgur.com/v1KpXxR.png)

The details about `Yatia` design can be check from [official document](https://docs.bentoml.org/projects/yatai/en/latest/concepts/architecture.html#yatai)

Here introduce the _**Yatai webserver**_ as the blue part of architecture diagram.

### Yatai-System

#### Yatai Webserver
![](https://i.imgur.com/B5Th20f.png)

The Yatai webserver is install via helm chart proviede by Yatai offical repo. Including:
1. Dashboard interfacre for all comonent
2. Registry for managing models and bentos with version control.
3. APIs for programmatic access of Yatai functionality, e.g. bento push and pull.

The yatai install in K8S, affiliate with relational DB and abject store.

#### Relational DB (Here using PostgreSQL)

Storing metadata of models, bentos, and users. Any PostgreSQL compatible relational databases can be used with provided user name and password. A database named yatai will be created to store tables of various metadata.

> AWS / GCP cloud solution also available.
> ![](https://i.imgur.com/AI7WPg1.png)

#### Object Store (Here using Minio)
Bucket storage for storing model and bento object. 
>Any S3 compatible services can be used providing the bucket name and endpoint address.
>![](https://i.imgur.com/0mIaf9j.png)

### Yatai-Image-Builder

* Build OCI images for bentos.
* Generate Bento CR

An add-on on top of `yatai` for building OCI images for bentos.
The `yatai-image-builder` runs in K8S, it is the operator of **BentoRequest CRD**, it is responsible for reconcile **BenroRequest CR** and then build the image for Bento, after the image is built Bento CR is generated, `yatai-deployment` component will depend on **Bento CR** to deploy Bento in K8S.

The **Docker Registry** stores the OCI images built from bentos. `yatai-deployment` fetches images from the OCI image registry during deployment.

### Yatai-Deployment

Deploy bentos in K8S. An add-on on top of `yatai` for deploying bentos to Kubernetes.
It is the operator of **BentoDeployment CRD**, it is responsible for reconcile **BentoDeployment CR**


## Install Yatai

#### preparing minikube environment

1. Start Minikube with kubernetes v1.22
`minikube start --cpus 8 --memory 16384 --driver=docker --kubernetes-version=v1.22.0`
> The releases of kubernetes version https://github.com/kubernetes/minikube/releases?page=2 

![](https://i.imgur.com/i2r1eUT.png)

2. installing
* yatai system : https://docs.bentoml.org/projects/yatai/en/latest/installation/yatai.html
* yatai image builder : https://docs.bentoml.org/projects/yatai/en/latest/installation/yatai_image_builder.html
* yatai deployment : https://docs.bentoml.org/projects/yatai/en/latest/installation/yatai_deployment.html
* premethus: https://docs.bentoml.org/projects/yatai/en/latest/observability/metrics.html

![](https://i.imgur.com/O3R7Mdy.png)



### Check the deployment pod by k9s

![](https://i.imgur.com/Oh7vDfW.png)


## Setup Yatai Dashboard Account.

Following the quick-tour from official github page
https://github.com/bentoml/Yatai#quick-tour

1. forwarding the 80 port of `yatai-system` to 8080 on the host, let others to access. 
```
kubectl --namespace yatai-system port-forward svc/yatai 8080:80
```
2. Due to I running the minikube on the remote server. Use http tunnel to forward the traffic from local browser.
```
ssh -i ".ssh/lab_ben.pem" -N -p 22 ubuntu@ec2-13-112-207-201.ap-northeast-1.compute.amazonaws.com -L 127.0.0.1:8080:127.0.0.1:8080
```
3. get the `secret` of `yatai-system` to get the `.data.YATAI_INITIALIZATION_TOKEN`, which is using for first log in and create the account.
```
YATAI_INITIALIZATION_TOKEN=$(kubectl get secret yatai-env --namespace yatai-system -o jsonpath="{.data.YATAI_INITIALIZATION_TOKEN}" | base64 --decode)
echo "Open in browser: http://127.0.0.1:8080/setup?token=$YATAI_INITIALIZATION_TOKEN"
```
i.e. you will get something like this
> Open in browser: http://127.0.0.1:8080/setup?token=lJ3Tka19W0WdhkJy
::: info 
:bulb: **HINT**: replace the token to yours own. 
:::

![](https://i.imgur.com/VCU37dH.png)

sign in the user account.

![](https://i.imgur.com/0jsQIgb.png)

The Yatai-Dashboard can be used.

![](https://i.imgur.com/CjRXkhj.png)

## log in Yatai from terminal

To push bento (Packed ML model for serving) to Yatai from bentoml cli. Have to log in yatai from terminal first.

For log in Yatai, create API token from here.

http://127.0.0.1:8080/api_tokens

![](https://i.imgur.com/6WS1lQh.png)

![](https://i.imgur.com/bDjm6pg.png)

![](https://i.imgur.com/mzp2ZNA.png)

Copy the token and used it to log in from terminal.

![](https://i.imgur.com/njoDvZm.png)

![](https://i.imgur.com/kQs5obv.png)

#### check and push the bento to Yatai

![](https://i.imgur.com/VB3EdWH.png)

![](https://i.imgur.com/5D56DG2.png)

Here we can check the pushed bento from Yatai Dashboard

![](https://i.imgur.com/803z1J8.png)

## Deploy the bento SVC to minikube.

Here we install `yatai-image-build` and `yatai-deployment`. Which these two component is responsible for build docker image from `bento` and then deploy to minikube.

The following operation is easy from GIU.

![](https://i.imgur.com/quCsDS8.png)

![](https://i.imgur.com/kM8Cby7.png)

![](https://i.imgur.com/hjP3TDC.png)

set up the desired configuration for deployment and commit.

![](https://i.imgur.com/tdSOf3U.png)

## Monitoring
![](https://i.imgur.com/NYmOw8j.png)

![](https://i.imgur.com/TJRt6Xx.png)

![](https://i.imgur.com/5bxdmMN.png)

![](https://i.imgur.com/xiOboD5.png)

![](https://i.imgur.com/tILUdNC.png)


![](https://i.imgur.com/amxW8DT.png)

![](https://i.imgur.com/7JIZZI3.png)

## Accessing Bentoml Serving Endpoint

The Bento ML model serving pod is create and the service is defined.
The Minikube ingress will take care of the inbound traffic.

1. Check the Ingress
`kubectl get ingress -A`
![](https://i.imgur.com/lk4vAES.png)

2. Check the Service list
`minikube service list`
![](https://i.imgur.com/fONLuEL.png)
The ingress is running at <minikube internal ip>:30882

3. Edit the `/etc/hosts` and add the `domain name` of service's `Host` shown in the 1st step.
![](https://i.imgur.com/sPTKgFA.png)

4. Send the testing request to ingress from remote host.
    
![](https://i.imgur.com/47t42yd.png)

5. Build the ssh tunnel from local laptop.
`ssh -i "<CA>" -N -p 22 <user>@<remote server> -L 127.0.0.1:30882:192.168.49.2:30882`

6. Add the domain name to `127.0.0.1` from the laptop's `\etc\hosts`
    
Test the model inference reqeuest from laptop.
    
![](https://i.imgur.com/1V2rPcm.png)
