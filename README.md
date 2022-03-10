# prometheus-in-kubernetes

## What is Prometheus?

Prometheus is a monitoring solution for recording and processing any purely numeric time-series.

It collects metrics from configured targets at given intervals, evaluates rule expressions, displays the results, and can trigger alerts when specified conditions are observed.

![image](https://user-images.githubusercontent.com/37524392/157591634-0225fdb6-b1df-4347-bbc8-4424093e78cf.png)
> image source: https://prometheus.io/docs/introduction/overview/


## Installation of Prometheus

There are various methods for installing prometheus such as from binary, docker or directly from the source. Here, let me explain how we can deploy prometheus in kubernetes.

Prometheus Docker Images are readily available in Docker hub. You can run this docker image locally to test the image.

```
docker run \
    -p 9090:9090 \
    -v /localpath/to/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
```

To deploy prometheus in GKE, we have to take care of two things —

- **Configuration and Rules**

We can maintain configuration and rules in Kubernetes as ConfigMaps. Then mount this ConfigMap in the respective location of the containers.

Config map would look like this :

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ConfigMap Name}}
  namespace: {{Namespace Name}}
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    alerting:
      alertmanagers:
        - static_configs:
            - targets:
              # - alertmanager:9093
    rule_files:
      # - "first_rules.yml"
      # - "second_rules.yml"
    scrape_configs:
      - job_name: "prometheus"
        static_configs:
          - targets: ["localhost:9090"]
```

Another approach is to create a custom docker image with the rulesand configs and deploy this image in K8. This way we maintain the configs in the image rather than in ConfigMaps.

Since there are multiple rules and configs are quite big, I personally chose the later approach of creating custom image.

Here we keep the rules, Config and Dockerfile in our source code repository. Docker file would look like this.

```
FROM prom/prometheus:v2.22.1
RUN mkdir -p /etc/prometheus/rules
ADD prometheus.yml /etc/prometheus/
ADD rules /etc/prometheus/rules
```

- **Persistent disk for data retention**

Since Prometheus stores data in disk, we will need to mount a persistent disk to the k8 pod so that data will be retained even if there is a pod restart.

For this we use StorageClasses and Persistent Volume Claim. You may refer the following as an example —
https://github.com/sajankgit/prometheus-in-kubernetes/tree/main/Manifests

For scalability, we should be using NFS persistent disk instead of normal PD. NFS persistent disk will allow mounting disk with multiple pods. However, for the documentation purpose I am using normal PD.

## Deploying Prometheus in k8

1. Building Docker Image.

```
git clone https://github.com/sajankgit/prometheus-in-kubernetes.git
cd Docker
docker build -t prometheus .
```

2. Push this docker image to your repository

```
docker tag prometheus <docker-repo-url>/<path>:<tag>

docker push <docker-repo-url>/<path>:<tag>
```

3. Deploy in k8s

```
git clone https://github.com/sajankgit/prometheus-in-kubernetes.git

kubectl apply -f Manifests/
```

## Conclusion

Deploying prometheus in K8 will provide stability and scalability.

The repo which I shared are just sample files. You should be making changes according to your needs.

Hope this article helps you in case you are deploying prometheus in k8 ! Happy monitoring !


