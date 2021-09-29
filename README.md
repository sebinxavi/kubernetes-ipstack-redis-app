# kubernetes-ipstack-redis-app
This is an example of an simple Redis deployment with 4 replicas and selector label app=redis-app. The deployment has PodAntiAffinity configured to ensure the scheduler does not co-locate replicas on a single node.


In a three node cluster, a web application has in-memory cache such as redis. We want the web-servers to be co-located with the cache as much as possible.

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
