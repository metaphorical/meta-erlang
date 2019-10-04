# Running locally

## Just run it (local or Docker)

If you have everything brewed into your OS:

```
make
make run
```

Better choice is using Docker:

```
docker build . -t erlc
docker run -p 8080:8080 -it --name erlc erlc
```

> When you were doing some building locally and now wanna build it in docker, you need to remove fe directories first.

> To make things easier for myself and avoid having to maintain same list twice, I made small script:
> ```
> ./dev_pre_build.sh
> ```

Cleanup docker to be able to rebuild and rerun:

```
docker stop erlc | docker rm erlc
```

Kill **all** Erlang processes(in case you were running it not containerized and it got messy):

```
for i in `ps -ef | grep erl | awk '{print $2}'`; do echo $i; kill -9 $i; done
```


## Kubernetes

> These are general notes that will help you understand environment setup little more in depth, assuming that you understand basic k8s stuff.

### Minikube setup

To setup k8s locally you will ideally use [**minikube**](https://kubernetes.io/docs/setup/minikube/).

If you are om MacOS, here is [useful article](http://rastko.tech/kubernetes/2019/01/01/minikube-on-mac.html) i wrote that should help you go through kinks of installation.

So, let's get going...

#### Rough reality of container registry usage (in local)

You need to build your docker image and push it to container registry in order for deployment to work as it would out there.

##### Local, easier option
> But missing setup/usage of real container registry so you would lack that experience when you move forward.

**Here is it step by step:**

* Get your docker context into your minikube cluster

```
$ eval $(minikube docker-env)
```

> you can later revert this by using
>
> ```
> $ eval $(docker-machine env -u)
> ```


* While in cluster context setup local docker registry

```
$ docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

This should setup internal docker registry on *localhost:5000* so you can build and prepare images with (<> marks where your values go):

```
$ docker build . -t <your_tag>
$ docker tag <your_tag> localhost:5000/<your_tag>:<version>
```

* At this point you can **use localhost:5000/<your_tag>:<version>** as image in your deployment and that is it.

##### Proper, with real container registry
> Can also be used for custom/non-cloud kubernetes deployments (if those still exit somewhere at the time of reading).

Easiest way to set local test is using either docker container registry, aws ecr, quay.io or google container registry.

There are tutorials on the internet to setup local container registry, but then you need to hack around certificates etc so... swallow some upload time or find mentioned tuts.

To use one of these container registries, you can simply use minikube addon called **registry-creds**.

> For non mini kube install you can go to [registry creds](https://github.com/upmc-enterprises/registry-creds) repo and clone it, heads up tho - it is not too frequently maintained and docs are terrible.

```
$ minikube addons configure registry-creds
$ minikube addons enable registry-creds
```

> Make sure that, if you are setting it for AWS ECR, and you do not have role arn you want to use (you usually won't have and it is optional), you set it as something random as "changeme" or smt... It requires value, if you just press enter (since it is optional) deployment of creds pod will fail and make your life miserable.

In case of AWS ECR, that will let you pull from your repo directly setting url as container image and adding pull secret named **awsecr-cred**:
```
imagePullSecrets:
      - name: awsecr-cred
```

#### Ingress

```
$ minikube addons enable ingress
```

Make sure that you setup ingress based on your local hosts. For me, for example, it means adding following lines to /etc/hosts:

```
[minikube ip] microserver.erlang microserver.elixir microserver.c
```
Where **"[minikube ip]"** should be replaced with actual minikube ip.

Here is shortcut to do it:

```
$ echo "$(minikube ip) microserver.erlang" | sudo tee -a /etc/hosts
```

### Helm

When you are setting up stuff, you will need helm so, have minikube on and:

```
$ brew install kubernetes-helm
$ helm init
```
Valuable read: [https://docs.helm.sh/using_helm/](https://docs.helm.sh/using_helm/)

## Kubernetes deployment

### Project helm chart

> At this moment helm chart deploys one application to kubernetes with ingress and backend it needs. There is no out-of-the-box ability to run multiple apps on same ingress - I will probably add that later.

This project provides global template to run microserver apps in kubernetes in form of a helm chart.

Every app needs *values file* for general app deployment properties.

When you setup desired deployment properties for your app you can deploy it to k8s using:

Create:
```
helm upgrade -i --force microserver-erlang ./deployment/ --values ./erlang/deployment/values.yaml
```
delete:
```
helm delete microserver-erlang --purge
```