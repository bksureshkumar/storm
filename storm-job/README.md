# Storm

## Installing Apache Storm on Single Node

This document is for the Shared Service team for the installation of Apache Storm on the Dataproc cluster by using the Zookeeper from the Dataproc. Added the Single node installation for the development purposes.

### Installing Apache Storm on Single Node
https://storm.apache.org/releases/1.2.3/Setting-up-a-Storm-cluster.html

1. Create Compute Instance, with the tag (example: storm, used to create Firewall rules later)

gcloud beta compute --project=shared-service-dataproc instances create storm --zone=us-central1-a --machine-type=n1-standard-32 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=757523358990-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/cloud-platform --tags=storm,http-server,https-server --image=debian-9-drawfork-v20191004 --image-project=eip-images --boot-disk-size=110GB --boot-disk-type=pd-standard --boot-disk-device-name=storm --reservation-affinity=any

2. SSH into the storm Instance

3. Install Java and Set JAVA_HOME

sudo apt-get update
sudo apt-get install default-jre
sudo apt-get install default-jdk
Update Java Home:
sudo update-alternatives --config java
JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
source /etc/environment
echo $JAVA_HOME

4. Install Zookeeper

wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz
sudo tar -xvf zookeeper-3.4.9.tar.gz
mv zookeeper-3.4.9 zookeeper
sudo mkdir -p zookeeper/data
sudo nano zookeeper/conf/zoo.cfg

5. Configure ZooKeeper by adding the following to the file:

tickTime=2500
dataDir=~/zookeeper/data
clientPort=2181
maxClientCnxns=80

6. Start the ZooKeeper

cd zookeeper
sudo bin/zkServer.sh start

7. Install Storm

wget http://mirrors.sonic.net/apache/storm/apache-storm-1.2.3/apache-storm-1.2.3.tar.gz
tar -xvf apache-storm-1.2.3.tar.gz
mv apache-storm-1.2.3 storm
cd storm
mkdir data

8. Configure Storm

nano conf/storm.yaml
storm.zookeeper.servers:
 - "localhost"
storm.local.dir: “~/storm/data”
nimbus.host: "localhost"
supervisor.slots.ports:
 - 6700
 - 6701
 - 6702
 - 6703

9. Open the Firewall port for the UI Application to access

gcloud compute firewall-rules create storm-allow-tcp-8080 --source-ranges 0.0.0.0/0 --target-tags storm --allow tcp:8080


10. Start the Nimbus:
bin/storm nimbus &

11. Start the Supervisor:
bin/storm supervisor &

12. Start the UI
bin/storm ui &

13. Open the Browser and use the Public IP of the instance with port 8080
http://INSTANCE_PUBLIC_IP:8080

Installing Apache Storm on Dataproc Cluster
https://storm.apache.org/releases/1.2.3/Setting-up-a-Storm-cluster.html

1. Create Dataproc Cluster, with 3 Master and 3 Worker Nodes(Change the worker count based on need). The Master nodes have ZooKeeper already installed by the Dataproc. 

REGION=us-central1
ZONE=us-central1-a
CLUSTER_NAME=storm-cluster
gcloud dataproc clusters create ${CLUSTER_NAME} \
    --image-version 1.4.19-debian9 \
    --region ${REGION} \
    --zone ${ZONE} \
    --num-masters 3 \
    --num-workers 3 

2. Install Storm on all the 3 Worker Nodes and create a data path.

wget http://mirrors.sonic.net/apache/storm/apache-storm-1.2.3/apache-storm-1.2.3.tar.gz
tar -xvf apache-storm-1.2.3.tar.gz
mv apache-storm-1.2.3 storm
cd storm
mkdir data


3. Configure Storm on all the 3 Worker Nodes. The storm.zookeeper.servers are the IP address of the Dataproc Master Nodes, which has Zookeeper running already by Dataproc. Pick Worker node 0 as the nimbus.host and set the IP address of the Worker 0 to nimbus.host.

nano conf/storm.yaml
storm.zookeeper.servers:
 - "10.128.0.55"
 - "10.128.0.53"
 - "10.128.0.57"
storm.zookeeper.port: 2181 
storm.local.dir: "~/storm/data"
nimbus.host: "10.128.0.56"
supervisor.slots.ports:
 - 6700
 - 6701
 - 6702
 - 6703

4.Start the Nimbus on the Worker Node 0.

bin/storm nimbus &

5. Start the Supervisor on Worker 1& Worker 2.

bin/storm supervisor &

6. Start Storm UI in Nimbus Node.

bin/storm ui &

7. Add a Network Tag on the Nimbus node

gcloud compute instances add-tags storm-cluster-w-0 --tags=storm-cluster --zone=us-central1-a

8. Open the Nimbus node Firewall port for the UI 

gcloud compute firewall-rules create storm-cluster-allow-tcp-8080 --source-ranges 0.0.0.0/0 --target-tags storm-cluster --allow tcp:8080

9. Open the browser and type http://NIMBUS-IP:8080

http://35.224.117.123:8080
