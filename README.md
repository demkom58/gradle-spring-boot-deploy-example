# Target
Target of this repository is show how to build spring boot 
application Docker image and deploy it with kind and k8s at least 
in local.

## Prerequisites

We will need next tools installed:
1. JDK 8+
2. [Docker](https://www.docker.com/get-started)
3. [Kubernetes](https://kubernetes.io/docs/tasks/tools/)
4. [Kind](https://kind.sigs.k8s.io/)

## Project

This is simple rest application that response with 'Hello World!'
on root rout. To get try you can clone it with:
```
git clone https://github.com/demkom58/gradle-spring-boot-deploy-example.git
```
or
```
git clone git@github.com:demkom58/gradle-spring-boot-deploy-example.git
```

## Build

We can build Docker image using Cloud Native Buildpacks
(Paketo Buildpacks) without writing any Dockerfile or 
Docker-compose.

To build image we need spring boot plugin. Here are few examples:

### Maven
```xml
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
```

### Gradle
```groovy
plugins {
    id 'org.springframework.boot' version '2.5.4'
}
```

It's included by default in [spring start](https://start.spring.io/) 
project generator. Now we can start Docker and enter command to 
build image:

Gradle                     | Maven
-------------------------- | --------------------------------
`./gradlew bootBuildImage` | `./mvnw spring-boot:build-image`

When execution will be done, we should see new image appeared in
`docker images` with name of project and current version of 
project in tag.

Open terminal and enter `docker images gradle-spring-boot-deploy-example:0.0.1-SNAPSHOT`
to get detailed info about built image. Result:
```
REPOSITORY                          TAG              IMAGE ID       CREATED        SIZE
gradle-spring-boot-deploy-example   0.0.1-SNAPSHOT   93abbee19f85   41 years ago   289MB
```

To check that everything build right we can run our image and check
out application. Also, just for fun we can redirect port 8080 to 80:
`docker run --name gradle-spring-boot-deploy-example -p 
80:8080 gradle-spring-boot-deploy-example:0.0.1-SNAPSHOT`

Go to `localhost` or `localhost:80` in your web browser and check 
that everything works correctly, we should get in response:
```
Hello World!
```

We can stop container using `Ctrl + C` and remove container using
`docker rm gradle-spring-boot-deploy-example`.

## Deploy

Let's start Kubernetes cluster using kind:
```
kind create cluster
```

We need say K8s to load Docker image from registry like that:
```
kind load docker-image gradle-spring-boot-deploy-example:0.0.1-SNAPSHOT
```

To create deployment in k8s open terminal and enter next command:
```
kubectl create deployment example-app --image=gradle-spring-boot-deploy-example:0.0.1-SNAPSHOT
```

Now we can use `kubectl get deployment` to get deployments and
`kubectl get pod` to get pods.

```
> kubectl get deployment
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
example-app   1/1     1            1           44s

> kubectl get pod
NAME                           READY   STATUS    RESTARTS   AGE
example-app-584b66b696-q6ggs   1/1     Running   0          46s
```

Now we need make application in k8s accessible from outside. 
Using next command we will expose our deployment's port 8080:
```
kubectl expose deployment example-app --type=ClusterIP --name=gradle-spring-boot-deploy-example --port=8080
```

We can check created services with `kubectl get svc`:
```
> kubectl get svc
NAME                                TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
gradle-spring-boot-deploy-example   ClusterIP   10.96.58.78   <none>        8080/TCP    34s
kubernetes                          ClusterIP   10.96.0.1     <none>        443/TCP   25m
```

Now let's configure port forwarding like with Docker, execute:
`kubectl port-forward service/gradle-spring-boot-deploy-example 801:8080`

Open `localhost:801` in web browser, and we should se same 
response that have with docker only :)

To terminate port forwarding we can press `Ctrl + C`

## Clear workspace

1. To delete service enter: `kubectl delete svc gradle-spring-boot-deploy-example`
2. To delete deployment enter: `kubectl delete deployment example-app`
3. To delete cluster enter: `kind delete cluster`