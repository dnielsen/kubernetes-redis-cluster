# Create a Redis Cluster using Kubernetes on GCloud

### Requirements 
```
Install Python 2.7
Install the GCloud CLI [https://cloud.google.com/sdk "Google Cloud SDK"]
Log into https://console.cloud.google.com and navigate to the console.
Git clone this repo
```


### Install the GCloud CLI and Set Your Project
```
gcloud auth login // found in ./google-cloud-sdk/bin/gcloud init
gcloud init // Pick a gcloud project-id to use, or create a new inique project-id. Example: "devopsdays_dcn_01" 
gcloud config list project  
gcloud config set project <project-id>
gcloud init <project-name>

```

### Install the Kubernetes CLI
```
gcloud components install kubectl
gcloud components update kubectl
```

# Kubernetes Redis Cluster

### Create Container Instances
```
gcloud container clusters create <cluster-name> --zone <zone> // example: cluster-name="workshop01"; zone="us-west1-a". You might need to enable the API in the dashboard (ex: https://console.cloud.google.com/apis/api/container.googleapis.com/overview?project=devopsdays01)
// get available IP address range from your gcloud cluster
gcloud container clusters describe <cluster-name> --zone <zone>| grep servicesIpv4Cidr
// update clusterIP: "?.?.?.?" in services files (ex: redis-1.yaml)

```

### Create Disks
```
gcloud compute disks create --size=10GB \
  'redis-1' 'redis-2' 'redis-3' \
  'redis-4' 'redis-5' 'redis-6'
```
  
### Create Redis Cluster Configuration

```
kubectl create configmap redis-conf --from-file=redis.conf
```

### Create Redis Nodes

```
kubectl create -f replicasets
```

### Create Redis Services

```
kubectl create -f services
```

### Connect Nodes

```
kubectl run -i --tty ubuntu --image=ubuntu \
  --restart=Never /bin/bash
```

```
apt-get update
apt-get install ruby vim wget redis-tools
gem install redis
wget http://download.redis.io/redis-stable/src/redis-trib.rb
chmod 744 redis-trib.rb
```

```
// update with ip addresses used in services files above
./redis-trib.rb create --replicas 1 \
  10.131.242.1:6379 \
  10.131.242.2:6379 \
  10.131.242.3:6379 \
  10.131.242.4:6379 \
  10.131.242.5:6379 \
  10.131.242.6:6379
```

### Add a new node

```
gcloud compute disks create --size=10GB 'redis-7'
```

```
// create redis-7.yaml replicaset file
kubectl create -f replicasets/redis-7.yaml
```

```
// create redis-7.yaml services file
kubectl create -f services/redis-7.yaml
```

```
// get name of your pod (pick any of the redis cluster instances. ex: redis-1-..)
kubectl get pods

// connect to a redis instance (aka pod?) in your cluster
kubectl exec -i --tty <pod name> redis-cli

// update the ip address
CLUSTER MEET 10.131.242.7 6379
```
### Open Port to access Redis Cluster from outside of the Cluster
```
tbd
```
