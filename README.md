# Ansible scripts to deploy cri-o container runtime in K8s running on Debian 10, Flannel and KVM

I'm using KVM cluster deployed using jouros/Terraform-libvirt-k8s scripts


### Deploy 

ansible-playbook master-playbook-2.yml
ansible-playbook node-playbook.yml


#### Some checks

crio --version
crio version 1.19.0
Version:       1.19.0
GitCommit:     1aea4546834f909fc277e9ec72f00d032c2440be
GitTreeState:  dirty
BuildDate:     2020-12-05T15:19:41Z
GoVersion:     go1.15.2
Compiler:      gc
Platform:      linux/amd64
Linkmode:      dynamic

crictl --runtime-endpoint unix:///var/run/crio/crio.sock version
Version:  0.1.0
RuntimeName:  cri-o
RuntimeVersion:  1.19.0
RuntimeApiVersion:  v1alpha1

sudo curl -v --unix-socket /var/run/crio/crio.sock http://localhost/info | jq
{
  "storage_driver": "overlay",
  "storage_root": "/var/lib/containers/storage",
  "cgroup_driver": "systemd",
  "default_id_mappings": {
    "uids": [
      {
        "container_id": 0,
        "host_id": 0,
        "size": 4294967295
      }
    ],
    "gids": [
      {
        "container_id": 0,
        "host_id": 0,
        "size": 4294967295
      }
    ]
  }
}

crio-status info
cgroup driver: systemd
storage driver: overlay
storage root: /var/lib/containers/storage
default GID mappings (format <container>:<host>:<size>):
  0:0:4294967295
default UID mappings (format <container>:<host>:<size>):
  0:0:4294967295


### Deploy Busybox

$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
busybox   1/1     Running   0          91m

Busybox is running on node worker4:
$ kubectl describe pod busybox
...
Node:         worker4


### Crictl tool

Crictl is admin tool to inspect and debug container runtimes and applications on a Kubernetes node

worker4: crictl pods
POD ID              CREATED             STATE               NAME                    NAMESPACE           ATTEMPT
8f493881657c2       15 minutes ago      Ready               busybox                 default             0
ce124b5f6b042       18 minutes ago      Ready               kube-proxy-zxkxr        kube-system         0
03a2b8874b9eb       18 minutes ago      Ready               kube-flannel-ds-2dbqx   kube-system         0

E.g. Inspect busybox network:

crictl inspectp 8f493881657c2
...
"io.kubernetes.cri-o.CNIResult": "{\"cniVersion\":\"0.4.0\",\"interfaces\":[{\"name\":\"cni0\",\"mac\":\"fe:c7:2a:76:21:49\"},{\"name\":\"vethdac02e7f\",\"mac\":\"d2:e2:5e:30:a3:8f\"},{\"name\":\"eth0\",\"mac\":\"32:dd:5d:fc:58:2c\",\"sandbox\":\"/var/run/netns/53d7b9f6-4634-404c-80fc-2ba74c9961c9\"}],\"ips\":[{\"version\":\"4\",\"interface\":2,\"address\":\"10.244.4.3/24\",\"gateway\":\"10.244.4.1\"}],\"routes\":[{\"dst\":\"10.244.0.0/16\"},{\"dst\":\"0.0.0.0/0\",\"gw\":\"10.244.4.1\"}],\"dns\":{}}",
...



