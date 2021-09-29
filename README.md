# Kubernetes Ipstack Redis App

Here is an example of an simple web application which has a in-memory cache such as redis Redis deployment with 4 replicas and selector label app=redis-app in a three node kubernetes cluster. 

We want the web-servers to be co-located with the cache as much as possible.

Here is the yaml snippet of a simple redis deployment with four replicas and selector label app=redis-app. The deployment has PodAntiAffinity configured to ensure the scheduler does not co-locate replicas on a single node.

~~~
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-app
  labels:
    app: redis-app

spec:

  replicas: 4
  selector:
    matchLabels:
      app: redis-app

  template:

    metadata:
      labels:
        app: redis-app

    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - redis-app
            topologyKey: "kubernetes.io/hostname"

      containers:

        - name: redis
          image: redis:latest
          ports:
            - containerPort: 6379
~~~


The below yaml snippet of the webserver deployment has podAntiAffinity and podAffinity configured. This informs the scheduler that all its replicas are to be co-located with pods that have selector label app=redis-app. This will also ensure that each web-server replica does not co-locate on a single node.


~~~
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ipstackapp
  labels:
    app: ipstackapp
spec:

  replicas: 6
  selector:
    matchLabels:
      app: ipstackapp

  template:
    metadata:
      labels:
        app: ipstackapp

    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                  - ipstackapp
            topologyKey: "kubernetes.io/hostname"
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                  - redis-app
            topologyKey: "kubernetes.io/hostname"

      containers:

        - name: ipstackapp
          image: fujikomalan/ipstack:latest
          ports:
            - containerPort: 8080
          envFrom:
            - configMapRef:
                name: ipstackapp
~~~
