Install microk8s

Enable addons: `rbac dns registry storage`
- [microk8s - Working with image registries](https://microk8s.io/docs/working)

```
sudo microk8s.enable dns rbac storage registry
```

Add microk8s image registry to local docker daemon config for insecure registries.
Add `localhost:32000` to `/etc/docker/daemon.json` and restart docker.

```
sudo vi /etc/docker/daemon.json
sudo service docker restart
```

### Troubleshooting pod network connectivity...

> The Kubenet network plugin used by MicroK8s creates a cbr0 interface when the first pod is created. If you have ufw enabled, you’ll need to allow traffic on this interface:
> - [microk8s - Troubleshooting](https://microk8s.io/docs/)
>
> ```
> sudo ufw allow in on cbr0 && sudo ufw allow out on cbr0
> ```
> 
> Make sure packets to/from the pod network interface can be forwarded to/from the default interface on the host via the iptables tool. Such changes can be made persistent by installing the iptables-persistent package:
> 
> ```
> sudo iptables -P FORWARD ACCEPT
> sudo apt-get install iptables-persistent
> ```
> 
> or, if using ufw:
> 
> ```
> sudo ufw default allow routed
> ```

Install Helm

```
kubectl -n kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account tiller
```

Install Strimzi

```
helm repo add strimzi http://strimzi.io/charts/
helm install strimzi/strimzi-kafka-operator \
--name strimzi \
--namespace strimzi
```

Create Kafka Cluster

```
kubectl apply -f simple-strimzi.yaml
```


Publish kafka-lag-exporter image to microk8s docker registry

```
export DOCKER_REPOSITORY="localhost:32000"
sbt updateHelmChart docker:publish
```

TODO:

- create topic
- run producers and consumers with groups

