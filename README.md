# go-docker-kubernetes

In this post, we'll build a Docker image from the simple application we created earlier ([Web Application Part 0 (Introduction)](https://www.bogotobogo.com/GoLang/GoLang_Web.php)).

Here is the code:

![](https://www.bogotobogo.com/GoLang/images/Web-Docker-Image/app-code.png)

This code is available : [web-app-0.go](https://www.bogotobogo.com/GoLang/images/Web/web-app-0.go)

The application looks like this:

![](https://www.bogotobogo.com/GoLang/images/Web-Docker-Image/localhost-3000-index.png) ![](https://www.bogotobogo.com/GoLang/images/Web-Docker-Image/localhost-3000-health-check.png)

# Dockerfile

Here is our Dockerfile:

```
FROM golang:alpine

RUN mkdir /app

COPY . /app

WORKDIR /app

RUN go build -o main .

CMD ["/app/main"]
```

Let's build the image:

`$ docker build -t go-app-img .`

```
Sending build context to Docker daemon 7.304MB

Step 1/6 : FROM golang:alpine

---\> b97a72b8e97d

Step 2/6 : RUN mkdir /app

---\> Using cache

---\> 493029660590

Step 3/6 : COPY . /app

---\> e907c54b6b18

Step 4/6 : WORKDIR /app

---\> Running in 130a4c80c611

Removing intermediate container 130a4c80c611

---\> d7bb7a9a6db8

Step 5/6 : RUN go build -o main .

---\> Running in 518fd62e940b

Removing intermediate container 518fd62e940b

---\> 5ca1a1ef0e58

Step 6/6 : CMD ["/app/main"]

---\> Running in 675128cd77bf

Removing intermediate container 675128cd77bf

---\> 12af20a5db3f

Successfully built 12af20a5db3f

Successfully tagged go-app-img:latest
```

Run it:

`$ docker run -d -p **3333** :3000 --name go-app-container go-app-img`

```
57a96e20d1766978d25685dd86384d696ba8d64433ef70ca9f41d64dcaefa1d9
```

Here we have indicated Docker to run a new container from our image (go-app-img) binding host port  **3333**  to container's internal port  **3000**  (-p 3333:3001), running the container in detached mode (-d) and giving a custom name to this container (-name go-app-container).

Go to the browser and navigate to  **localhost:3333** :

![](https://www.bogotobogo.com/GoLang/images/Web-Docker-Image/localhost-3333-index.png)

# Deploying to Kubernetes cluster

Let's deploy our app to Kubernetes cluster.

Minikube **(Skip if you're use Kubernetes Dokcer Desktop)**

First, we need to start the minikube:

`$ minikube start`

```
😄minikube v1.0.0 on darwin (amd64)

🤹Downloading Kubernetes v1.14.0 images in the background ...

💡Tip: Use 'minikube start -p ' to create a new cluster, or 'minikube delete' to delete this one.

🔄Restarting existing virtualbox VM for "minikube" ...

⌛Waiting for SSH access ...

📶"minikube" IP address is 192.168.99.100

🐳Configuring Docker as the container runtime ...

🐳Version of container runtime is 18.06.2-ce

⌛Waiting for image downloads to complete ...

✨Preparing Kubernetes environment ...

🚜Pulling images required by Kubernetes v1.14.0 ...

🔄Relaunching Kubernetes v1.14.0 using kubeadm ...

⌛Waiting for pods: apiserver proxy etcd scheduler controller dns

📯Updating kube-proxy configuration ...

🤔Verifying component health ......

💗kubectl is now configured to use "minikube"

🏄Done! Thank you for using minikube!
```

Check the current status of the cluster:

`$ kubectl get svc`

```
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
kubernetes ClusterIP 10.96.0.1 \<none\> 443/TCP 12d
```

`$ kubectl get pods`

```
No resources found.
```

`$ kubectl get deployments`

```
No resources found.
```

Push the image to DockerHub

Let's push the image to DockerHub. To do that we may want to build the image specifying the repository:

`$ docker build -t dockerbogo/go-app .`

```
...
Successfully tagged dockerbogo/go-app:latest
```

`$ docker push dockerbogo/go-app:latest` **(Skip if you are using local docker image)**

```
The push refers to repository [docker.io/dockerbogo/go-app]
...
Deploy to minikube cluster
```

Here is our deploy manifest file ( **deployment.yaml** ):

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-go-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: go-app
  template:
    metadata:
      labels:
        app: go-app
    spec:
      containers:
      - name: go-app-container
        image: abyanjksatu/go-app
        imagePullPolicy: IfNotPresent
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 3000
```

Deploy:

`$ kubectl create -f deployment.yaml`

```
deployment.apps/my-go-app created
```

`$ kubectl get deployments`

```
NAME READY UP-TO-DATE AVAILABLE AGE
my-go-app 1/1 1 1 24s
```

`$ kubectl get pods`

```
NAME READY STATUS RESTARTS AGE
my-go-app-5bb8767f6d-2pdtk 1/1 Running 0 43s
```

## Exposing the go-app pod

Let's create a Service object that exposes the deployment:

`$ kubectl expose deployment my-go-app --type=NodePort --name=go-app-svc --target-port=3000`

```
service/go-app-svc exposed
```

`$ kubectl get svc`

```
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
go-app-svc **NodePort** 10.111.127.140 \<none\> 3000: **30652** /TCP 81s
kubernetes ClusterIP 10.96.0.1 \<none\> 443/TCP 12d
```

`$ minikube ip`

```
192.168.99.100
```

## Testing - access the app

Go to browser with  **minikube-ip:nodePort**  which is  **192.168.99.100:30652** :

![](https://www.bogotobogo.com/GoLang/images/Web-Docker-Image/go-app-via-nodePort.png) ![](https://www.bogotobogo.com/GoLang/images/Web-Docker-Image/go-app-via-nodePort-health_check.png)

Great! Now we've just deployed our go application to Kubernetes cluster.

Source:
- https://www.bogotobogo.com/GoLang/GoLang_Web_Building_Docker_Image_and_Deploy_to_Kubernetes.php
- https://blog.raulnq.com/running-local-images-in-kubernetes-docker-desktop