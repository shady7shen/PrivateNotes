- [Linux Preparation](#linux-preparation)
  - [change machine id](#change-machine-id)
  - [install dependencies](#install-dependencies)
  - [install docker](#install-docker)
  - [install kubernetes binaries](#install-kubernetes-binaries)
  - [setenforce util](#setenforce-util)
  - [install crictl (optional)](#install-crictl-optional)
  - [can be added to ~/.profile for proxy setting](#can-be-added-to-profile-for-proxy-setting)
- [Start of Cluster Installation](#start-of-cluster-installation)
  - [init the cluster](#init-the-cluster)
  - [configure kubectl with default admin user](#configure-kubectl-with-default-admin-user)
  - [install calico (1)](#install-calico-1)
  - [prepare calico etcd certs](#prepare-calico-etcd-certs)
  - [install calico (2)](#install-calico-2)
  - [kubernetes api server parameters is located in](#kubernetes-api-server-parameters-is-located-in)
- [Post Installation](#post-installation)
  - [install kubernetes dashboard](#install-kubernetes-dashboard)
  - [prepare a user to be used with kubernetes dashboard](#prepare-a-user-to-be-used-with-kubernetes-dashboard)
  - [generate the join command for cluster](#generate-the-join-command-for-cluster)
- [untaint the master node so that pod can be started on this node](#untaint-the-master-node-so-that-pod-can-be-started-on-this-node)
  - [kubectl logs showing Forbidden error](#kubectl-logs-showing-forbidden-error)
- [Appendix](#appendix)
  - [retrieve server ssl certificate](#retrieve-server-ssl-certificate)
  - [etcdctl check (do with care due to different etcd version locally and in kubernetes container)](#etcdctl-check-do-with-care-due-to-different-etcd-version-locally-and-in-kubernetes-container)
  - [install crictl tool](#install-crictl-tool)
  - [install hadoop client](#install-hadoop-client)
  - [add insecure docker registry server by creating/modifying `/etc/docker/daemon.json`](#add-insecure-docker-registry-server-by-creatingmodifying-etcdockerdaemonjson)
  - [some linux commands](#some-linux-commands)
  - [journal log](#journal-log)
  - [kubectl command collection](#kubectl-command-collection)
  - [delete Error pods in batch](#delete-error-pods-in-batch)
  - [SSL Certificate Preparation](#ssl-certificate-preparation)
    - [use openssl](#use-openssl)
      - [Example](#example)
    - [Use cfssl](#use-cfssl)
      - [Pre-requisite GO 1.8+](#pre-requisite-go-18)
      - [install cfssl](#install-cfssl)
      - [generate certificate / key pair signed by a CA](#generate-certificate--key-pair-signed-by-a-ca)
  - [Kubernetes Persistence Volume Template](#kubernetes-persistence-volume-template)
  - [busybox pod template](#busybox-pod-template)
  - [openjdk pod template](#openjdk-pod-template)
  - [docker registry pod template](#docker-registry-pod-template)
  - [hdfs yaml image](#hdfs-yaml-image)
  - [base64 decoding](#base64-decoding)
  - [base64 encoding](#base64-encoding)




# Linux Preparation
## change machine id
```
rm /etc/machine-id
rm /var/lib/dbus/machine-id
systemd-machine-id-setup
cp /etc/machine-id /var/lib/dbus/machine-id
```

## install dependencies
```
apt-get update && apt-get install -y apt-transport-https curl selinux-utils rpcbind nfs-kernel-server nfs-common libnfsidmap2
```

## install docker
```
apt-get update && apt-get install -y docker.io
```

## install kubernetes binaries
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl
```
Add cgroup-driver parameter to kubeadm config file
```
docker info | grep -i cgroup
```
kubeadm config file is located in
` /etc/systemd/system/kubelet.service.d/10-kubeadm.conf`
set parameter as `--cgroup-driver=<cgroupfs_value>`


## setenforce util
```
apt install selinux-utils
```
#/etc/fstab <-- comment out swap and restart

## install crictl (optional)
* https://github.com/kubernetes-incubator/cri-tools/releases
```
wget https://github.com/kubernetes-incubator/cri-tools/releases/download/v1.0.0-beta.1/crictl-v1.0.0-beta.1-linux-amd64.tar.gz
tar -xf crictl-v1.0.0-beta.1-linux-amd64.tar.gz
mv crictl /usr/bin
```

##install weave (optional)
```
sudo curl -L git.io/weave -o /usr/local/bin/weave
sudo chmod a+x /usr/local/bin/weave
```

## can be added to ~/.profile for proxy setting
```
HTTP_PROXY=http://proxy.van.sap.corp:8080
http_proxy=$HTTP_PROXY
HTTPS_PROXY=$HTTP_PROXY
https_proxy=$HTTP_PROXY
NO_PROXY=localhost,ub,10.160.205.56,10.96.0.0/12,172.17.0.0/16,192.168.0.0/16
no_proxy=$NO_PROXY
export HTTP_PROXY HTTPS_PROXY http_proxy https_proxy no_proxy NO_PROXY
setenforce 0
mesg n
```


# Start of Cluster Installation

## init the cluster
```
kubeadm init --pod-network-cidr=192.168.0.0/16
```

## configure kubectl with default admin user
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## install calico (1)
```
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/rbac.yaml
curl https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/calico.yaml -O
```

## prepare calico etcd certs
```
cp /etc/kubernetes/pki/etcd/ca.crt etcd-ca
cp /etc/kubernetes/pki/apiserver-etcd-client.crt etcd-cert
cp /etc/kubernetes/pki/apiserver-etcd-client.key etcd-key
kubectl delete secret calico-etcd-secrets2 -n kube-system
kubectl create secret generic calico-etcd-secrets2 -n kube-system --from-file=etcd-ca --from-file=etcd-cert --from-file=etcd-key
kubectl -n kube-system get secret calico-etcd-secrets2 -o yaml
```
in the output you can get the base64 secret that can be used in the calico.yaml file. Change the etcd endpoint value as well as the certificate values in the calico.yaml file.

## install calico (2)
kubectl apply -f calico.yaml


## kubernetes api server parameters is located in
`/etc/kubernetes/manifest/kube-apiserver.yaml`

# Post Installation

## install kubernetes dashboard
```
curl https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```
modify the yaml file,
```
#  args:       
#  - --default-cert-dir=/certs
#  - --tls-cert-file=server.crt
#  - --tls-key-file=server.key
# ...
#  volumes
#  - name: kubernetes-dashboard-certs
#    hostPath:
#      path: /etc/ssl/kdashboard/certs
#      type: Directory
```
Then apply the yaml file
```
kubectl apply -f kubernetes-dashboard.yaml
```
Expose the web interface
```
kubectl expose deployment kubernetes-dashboard -n kube-system --type NodePort --name dashboard-external
```

## prepare a user to be used with kubernetes dashboard 
```
==== shady-as-user.yaml ====
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
=============================
kubectl create -f shady-as-user.yaml
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep shady | awk '{print $1}')
# get the bearer token
```

## generate the join command for cluster
```
kubeadm token create --print-join-command

kubeadm join ub:6443 --token dbxare.qdjqfsg7jj42z7vk --discovery-token-ca-cert-hash sha256:453b08fd69906815e35bc38386645348c56d91e888a9aa5cd3e0bf148afcc856
```

# untaint the master node so that pod can be started on this node
```
kubectl taint node <master node name> node-role.kubernetes.io/master:NoSchedule-
kubectl taint nodes --all node-role.kubernetes.io/master-
```

## kubectl logs showing Forbidden error
grant cluster-admin role to system:anonymous user (do with care)
```
kubectl create clusterrolebinding cluster-system-anonymous --clusterrole=cluster-admin --user=system:anonymous
```

# Appendix
## retrieve server ssl certificate
```
openssl s_client -connect 10.160.210.78:10250 -showcerts
```
Add certificate to CA-certificate list `/usr/local/share/ca-certificates/`
```
sudo cp foo.crt /usr/local/share/ca-certificates/foo.crt
sudo update-ca-certificates
```


## etcdctl check (do with care due to different etcd version locally and in kubernetes container)
```
etcdctl --endpoints https://ub:2379 --cert-file /etc/kubernetes/pki/apiserver-etcd-client.crt --key-file /etc/kubernetes/pki/apiserver-etcd-client.key --ca-file /etc/kubernetes/pki/ca.crt ls

etcd --client-cert-auth=true --peer-client-cert-auth=true --data-dir=/var/lib/etcd --cert-file=/etc/kubernetes/pki/etcd/server.crt --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt --advertise-client-urls=https://127.0.0.1:2379 --key-file=/etc/kubernetes/pki/etcd/server.key --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt --peer-key-file=/etc/kubernetes/pki/etcd/peer.key --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt --listen-client-urls=https://127.0.0.1:2379
```

## install crictl tool
github.com/kubernetes-incubator/cri-tools/cmd/crictl
[WARNING HTTPProxy]: Connection to "https://10.160.203.5" uses proxy "http://proxy.van.sap.corp:8080". If that is not intended, adjust your proxy settings
```
wget https://github.com/kubernetes-incubator/cri-tools/releases/download/v1.0.0-beta.1/crictl-v1.0.0-beta.1-linux-amd64.tar.gz
tar -xf crictl-v1.0.0-beta.1-linux-amd64.tar.gz
mv crictl /usr/bin
```


## install hadoop client
grab zip file http://archive.apache.org/dist/hadoop/core/
unzip and make the folder part of PATH
```
core-site.xml
----------------
<configuration>
<property>
<name>fs.defaultFS</name>
<value>hdfs://10.160.210.77:30020</value>
</property>
</configuration>
```

## add insecure docker registry server by creating/modifying `/etc/docker/daemon.json`
```
{
  "insecure-registries": ["docker-registry:31200","localhost:31200"]
}
```


kubectl create clusterrolebinding cluster-system-anonymous --clusterrole=cluster-admin --user=kube-system:default
helm init --service-account=shady --upgrade

kubectl create clusterrolebinding weave-net-as-admin --clusterrole=cluster-admin --serviceaccount=kube-system:weave-net

## some linux commands
```
pstree -l -a -A 20708
tree -d my_container/
```

## journal log
```
journalctl -r | more
journalctl --vacuum-time=1s
```



## kubectl command collection
* port forwarding
```
kubectl port-forward vora-consul-1-0 8500:8500
```
* use ssh tunnel to allow external accesss
* add \\\n -ui to the commandline to provide the consul ui


## delete Error pods in batch
```
kubectl get pods -n vora | grep Error | cut -d ' ' -f 1 | xargs kubectl -n vora delete pod
```



"args" : ["/bin/bash", 
"-c", 
"--",
"\"while true; do sleep 30; done;\""]


"args": [
          "/usr/bin/python",
          "/dqp/scripts/start_service.py",
          "--consul-address",
          "vora-consul.vora:8500",
          "--address",
          "$(POD_IP)",
          "--trace-dir",
          "/var/local/vora/vora-dlog/$(NAMESPACE)",
          "--stderr-no-capture",
          "--readiness-port",
          "12345",
          "--port",
          "10002",
          "--node-port",
          "10003",
          "--trace-level",
          "info",
          "-c",
          "deregister_critical_service_after=\"2m\"",
          "--bind-address",
          "$(POD_IP)",
          "-c",
          "tail_store_size=\"4g\"",
          "vora-dlog"
        ]

http://aclisp.github.io/blog/2015/08/25/kubernetes-startup.html



## SSL Certificate Preparation
### use openssl
generate private key 
```
openssl genrsa -des3 -out server.key 2048
```
generate a certificate signing request
```
openssl req -new -key server.key -out server.csr -config openssl.cnf

openssl req -new -key server.key -subj "/CN=kube-etcd" -out server.csr
```
remove passphrase from key
```
openssl rsa -in server.key -out server.key
```
generate a self-signed certificate
```
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 19820528001 -out server.crt

openssl x509 -req -extfile <(printf "subjectAltName=DNS:example.com,DNS:www.example.com") -days 365 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt
```
#### Example
etcd certificates
```
openssl req -nodes -newkey rsa:2048 -keyout server.key -out server.csr -subj "/CN=kube-etcd"
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 19820528001 -out server.crt

openssl req -nodes -newkey rsa:2048 -keyout peer.key -out peer.csr -subj "/CN=kube-etcd-peer"
openssl x509 -req -days 3650 -in peer.csr -CA ca.crt -CAkey ca.key -set_serial 19820528002 -out peer.crt

openssl req -nodes -newkey rsa:2048 -keyout healthcheck-client.key -out healthcheck-client.csr -subj "/O=system:masters, /CN=kube-etcd-healthcheck-client"
openssl x509 -req -days 3650 -in healthcheck-client.csr -CA ca.crt -CAkey ca.key -set_serial 19820528002 -out healthcheck-client.crt
```
### Use cfssl
#### Pre-requisite GO 1.8+
```
add-apt-repository ppa:gophers/archive
apt-get update
apt-get install golang-1.10-go
```
#### install cfssl
```
go get -u github.com/cloudflare/cfssl/cmd/cfssl
```

#### generate certificate / key pair signed by a CA
```
echo '{ "key": { "algo": "rsa", "size":2048} }' | cfssl gencert -ca=ca.crt -ca-key=ca.key -cn="/CN=kube-etcd" -hostname="ub,localhost,10.160.205.56,127.0.0.1" - | cfssljson -bare server
mv server.pem server.crt && mv server-key.pem server.key

echo '{ "key": { "algo": "rsa", "size":2048} }' | cfssl gencert -ca=ca.crt -ca-key=ca.key -cn="/O=system:masters, /CN=kube-etcd-healthcheck-client" -hostname="ub,localhost,10.160.205.56,127.0.0.1" - | cfssljson -bare healthcheck-client
mv healthcheck-client.pem healthcheck-client.crt && mv healthcheck-client-key.pem healthcheck-client.key

echo '{ "key": { "algo": "rsa", "size":2048} }' | cfssl gencert -ca=ca.crt -ca-key=ca.key -cn="/CN=kube-etcd-peer" -hostname="ub,localhost,10.160.205.56,127.0.0.1" - | cfssljson -bare peer
mv peer.pem peer.crt && mv peer-key.pem peer.key

echo '{ "key": { "algo": "rsa", "size":2048} }' | cfssl gencert -ca=ca.crt -ca-key=ca.key -cn="/CN=kube-etcd-peer" -hostname="ub,localhost,10.160.205.56,127.0.0.1" - | cfssljson -bare apiserver-etcd-client
mv apiserver-etcd-client.pem ../apiserver-etcd-client.crt && mv apiserver-etcd-client-key.pem ../apiserver-etcd-client.key

echo '{ "key": { "algo": "rsa", "size":2048} }' | cfssl gencert -ca=ca.crt -ca-key=ca.key -cn="/CN=kube-etcd-peer" -hostname="ub2,localhost,10.160.193.201,127.0.0.1" - | cfssljson -bare kubelet
mv kubelet.pem kubelet.crt && mv kubelet-key.pem kubelet.key
```

## Kubernetes Persistence Volume Template
```
kind: PersistentVolume
apiVersion: v1
metadata:
  name: datadir-vora-consul-0
spec:
  capacity:
    storage: 25Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/storage/kubernetes/pv"
```
## busybox pod template
busybox.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
  nodeSelector:
    intRole: minion
```

## openjdk pod template
```
apiVersion: v1
kind: Pod
metadata:
  name: openjdk
spec:
  containers:
  - image: openjdk:10
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
    volumeMounts:
      - name: data
        mountPath: /root/data
  volumes:
    - name: data
      hostPath:
        path: /storage/kubernetes/hadoopcli
        type: Directory
  restartPolicy: Always
  nodeSelector:
    intRole: minion
```

## docker registry pod template 
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: docker-registry
  labels:
    app: docker
    type: registry
spec:
  replicas: 1
  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: docker
        type: registry
    spec:
      containers:
        - name: docker-registry
          image: registry:2
          command:
          - /bin/registry
          - serve
          - /etc/docker/registry/config.yml
          ports:
            - containerPort: 5000
          volumeMounts:
            - name: data
              mountPath: /var/lib/registry/
      volumes:
        - name: data
          hostPath:
            path: /storage/docker/registry
            type: Directory
      nodeSelector:
        intRole : master
```
## hdfs yaml image
```
apiVersion: v1
kind: Service
metadata:
  name: hdfs-namenode
spec:
  ports:
    - name: client
      port: 8020
  selector:
    app: hdfs
    type: namenode
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: hdfs-namenode
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: hdfs
        type: namenode
    spec:
      volumes:
        - name: data
          hostPath:
            path: /storage/kubernetes/hdfs/name
            type: Directory
      containers:
        - name: namenode
          image: uhopper/hadoop-namenode:2.7.2
          env:
            - name: CLUSTER_NAME
              value: hadoop
            - name: HDFS_CONF_dfs_namenode_datanode_registration_ip___hostname___check
              value: "false"
            - name: MULTIHOMED_NETWORK
              value: "0"
          ports:
            - containerPort: 50070
            - containerPort: 8020
          volumeMounts:
            - mountPath: /hadoop/dfs/data
              name: data
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: hdfs-datanode
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: hdfs
        type: datanode
    spec:
      volumes:
        - name: data
          hostPath:
            path: /storage/kubernetes/hdfs/data
            type: Directory
      containers:
        - name: datanode
          image: uhopper/hadoop-datanode:2.7.2
          env:
            - name: CORE_CONF_fs_defaultFS
              value: hdfs://hdfs-namenode:8020
            - name: HDFS_CONF_dfs_namenode_datanode_registration_ip___hostname___check
              value: "false"
          ports:
            - containerPort: 50075
            - containerPort: 50010
            - containerPort: 50020
          volumeMounts:
            - mountPath: /hadoop/dfs/data
              name: data
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
```
## base64 decoding
```
echo LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRFNE1EWXlOakE1TXpFek0xb1hEVEk0TURZeU16QTVNekV6TTFvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTUpTCmJzb09jQVdQTi8vbnFaMzh2WHV0SGxCMGwxK0hpTGFBVGpWWkx1TDlpVEZRWFdVdW05R2FZTE8zeFRXcEpPVFcKVkNGRTR1WmQ4WUpPWStLVVR6WGsvZytyYVZTa2pXZUlNUS9qeUQrWXFXTmlMVDZrVDBsRmdqM3lVQXUyTGZiUAo3Ym5DNXEvRksrQTBCVitXZzdEOS9xMktITXJHMHBPb1pHRHhLemZpT0wzTlo2ZTRPZFIvSzFEbUpIMjh0d2poClR6QUlrV0VFUXB3dllvVDRGS3ZzaDZLSk1VbHFGS3BpcFhBZmlXb3JwaUR4N2p5YzZsWDVEVk5VRlZYZHBVOEEKMkRzdHJvLzV3V1NhQ0pZaC9qWUdzUjdTdnFxRUhqWSsvMjBwY2JINXR1aG03QUVsTm9oRFF4SkpxbEs5ZXkvQQpkcUpkbEE4RjR6MWJqU09xL0tzQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFGNjE1ZGpLME9aMDhveWV4QmszRllTUU9vUkcKbmhmMTZrci9RTDZlQUpMUUlhSnBMQVJONE1rbFFXR3pkdCtnVzhiN2hzTHE0NGxyenRKR1NaTjVndlFOdTQzZwpxdmlESGswN0oxMzBqZ1ZiaHQ2NkpJRjF3TnE0NmR1dkRvdlQvY3ZBQjV5QU44QmVmdzdPdTFtUmJRdUdBVFJXCllRcU5vdzlCZ0k0Qllsb09kN3oyNkttUWt0dUQ5N29OK0hjMXE2UVE4dTd1akYvQWF0a25jdGFjUWJ2ckZweWkKZDM3YXM2aVRLRlZYR21DREpRTEoyR3BKMzdtTUl5QjBYK1Rwa3FEbHp0b3htVjVUTi9SM01pUDF0Y3k0U0dNcAozT0VrQndMV1dSeHpscXZvSUtJa28xbVJIbHpPRjNHSHBrZEh3d0pmR2tQdGQvelkxbFlNNHhFbkFLQT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo= | base64 -d
```
## base64 encoding
```
cat ca.crt | base64
```
