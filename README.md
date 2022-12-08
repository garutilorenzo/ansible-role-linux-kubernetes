[![GitHub issues](https://img.shields.io/github/issues/garutilorenzo/ansible-role-linux-kubernetes)](https://github.com/garutilorenzo/ansible-role-linux-kubernetes/issues)
![GitHub](https://img.shields.io/github/license/garutilorenzo/ansible-role-linux-kubernetes)
[![GitHub forks](https://img.shields.io/github/forks/garutilorenzo/ansible-role-linux-kubernetes)](https://github.com/garutilorenzo/ansible-role-linux-kubernetes/network)
[![GitHub stars](https://img.shields.io/github/stars/garutilorenzo/ansible-role-linux-kubernetes)](https://github.com/garutilorenzo/ansible-role-linux-kubernetes/stargazers)

# Install and configure a high available Kubernetes cluster

This ansible role will install and configure a high available Kubernetes cluster. This repo automate the installation process of Kubernetes using [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/).

This repo is only a example on how to use Ansible automation to install and configure a Kubernetes cluster. For a production environment use [Kubespray](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/)

## Requirements

Install ansible, ipaddr and netaddr:

```
pip install -r requirements.txt
```

Download the role form GitHub:

```
ansible-galaxy install git+https://github.com/garutilorenzo/ansible-role-linux-kubernetes.git
```

## Role Variables

This role accept this variables:

| Var   | Required |  Default | Desc |
| ------- | ------- | ----------- |  ----------- |
| `kubernetes_subnet`       | `yes`       |  `192.168.25.0/24` | Subnet where Kubernetess will be deployed. If the VM or bare metal server has more than one interface, Ansible will filter the interface used by Kubernetes based on the interface subnet |
| `disable_firewall`       | `no`       | `no`       | If set to yes Ansible will disable the firewall.   |
| `kubernetes_version`       | `no`       | `1.25.0`       | Kubernetes version to install  |
| `kubernetes_cri`       | `no`       | `containerd`       | Kubernetes [CRI](https://kubernetes.io/docs/concepts/architecture/cri/) to install.   |
| `kubernetes_cni`       | `no`       | `flannel`       | Kubernetes [CNI](https://github.com/containernetworking/cni) to install.  |
| `kubernetes_dns_domain`       | `no`       | `cluster.local`       | Kubernetes default DNS domain  |
| `kubernetes_pod_subnet`       | `no`       | `10.244.0.0/16`       | Kubernetes pod subnet  |
| `kubernetes_service_subnet`       | `no`       | `10.96.0.0/12`       | Kubernetes service subnet  |
| `kubernetes_api_port`       | `no`       | `6443`       | kubeapi listen port  |
| `setup_vip`       | `no`       | `no`       | Setup kubernetes VIP addres using [kube-vip](https://kube-vip.io/)   |
| `kubernetes_vip_ip`       | `no`       | `192.168.25.225`       | **Required** if setup_vip is set to *yes*. Vip ip address for the control plane  |
| `kubevip_version`       | `no`       | `v0.4.3`       | kube-vip container version  |
| `install_longhorn`       | `no`       | `no`       | Install [Longhorn](#longhorn), Cloud native distributed block storage for Kubernetes.  |
| `longhorn_version`       | `no`       | `v1.3.1`       | Longhorn release.  |
| `install_nginx_ingress`       | `no`       | `no`       | Install [nginx ingress controller](#nginx-ingress-controller)  |
| `nginx_ingress_controller_version`       | `no`       | `controller-v1.3.0`       | nginx ingress controller version  |
| `nginx_ingress_controller_http_nodeport`       | `no`       | `30080`       | NodePort used by nginx ingress controller for the incoming http traffic  |
| `nginx_ingress_controller_https_nodeport`       | `no`       | `30443`       |  NodePort used by nginx ingress controller for the incoming https traffic  |
| `enable_nginx_ingress_proxy_protocol`       | `no`       | `no`       | Enable  nginx ingress controller proxy protocol mode |
| `enable_nginx_real_ip`       | `no`       | `no`       | Enable nginx ingress controller real-ip module |
| `nginx_ingress_real_ip_cidr`       | `no`       | `0.0.0.0/0`       | **Required** if enable_nginx_real_ip is set to *yes* Trusted subnet to use with the real-ip module  |
| `nginx_ingress_proxy_body_size`       | `no`       | `20m`       | nginx ingress controller max proxy body size  |
| `sans_base`       | `no`       | `[list of values, see defaults/main.yml]`       | list of ip addresses or FQDN uset to sign the kube-api certificate  |

## Extra Variables

This role accept an extra variable *kubernetes_init_host*. This variable is used when the cluster is bootstrapped for the first time. The value of this variable must be the hostname of one of the master nodes. When ansible will run on the matched host kubernetes will be initialized.

## Cluster resource deployed

Whit this role [Nginx ingress controller](#nginx-ingress-controller) and [Longhorn](#longhorn) will be installed.

### Nginx ingress controller

[Nginx ingress controller](https://kubernetes.github.io/ingress-nginx/) is used as ingress controller.

The installation is the [bare metal](https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal-clusters) installation, the ingress controller then is exposed via a NodePort Service.
You can customize the ports exposed by the NodePort service, use [Role Variables](#role-variables) to change this values.

### Longhorn

[Longhorn](https://longhorn.io) is a lightweight, reliable, and powerful distributed block storage system for Kubernetes.

Longhorn implements distributed block storage using containers and microservices. Longhorn creates a dedicated storage controller for each block device volume and synchronously replicates the volume across multiple replicas stored on multiple nodes. The storage controller and replicas are themselves orchestrated using Kubernetes.

## Vagrant

To test this role you can use [Vagrant](https://www.vagrantup.com/) and [Virtualbox](https://www.virtualbox.org/) to bring up a example infrastructure. Once you have downloaded this repo use Vagrant to start the virtual machines:

```
vagrant up
```

In the Vagrantfile you can inject your public ssh key directly in the authorized_keys of the vagrant user. You have to change the *CHANGE_ME* placeholder in the Vagrantfile. You can also adjust the number of the vm deployed by changing the NNODES variable (Default: 6)

## Using this role

To use this role you follow the example in the [examples/](examples/) dir. Adjust the hosts.ini file with your hosts and run the playbook:

```
lorenzo@mint-virtual:~$ ansible-playbook -i hosts-ubuntu.ini site.yml -e kubernetes_init_host=k8s-ubuntu-0

PLAY [kubemaster] ***************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************
ok: [k8s-ubuntu-2]
ok: [k8s-ubuntu-1]
ok: [k8s-ubuntu-0]

TASK [ansible-role-kubernetes : include_tasks] **********************************************************************************************************************
included: /home/lorenzo/workspaces-local/ansible-role-kubernetes/tasks/setup_repo_Debian.yml for k8s-ubuntu-0, k8s-ubuntu-1, k8s-ubuntu-2 => (item=/home/lorenzo/workspaces-local/ansible-role-kubernetes/tasks/setup_repo_Debian.yml)

TASK [ansible-role-kubernetes : Install required system packages] ***************************************************************************************************
ok: [k8s-ubuntu-2]
ok: [k8s-ubuntu-1]
ok: [k8s-ubuntu-0]

TASK [ansible-role-kubernetes : Add Google GPG apt Key] *************************************************************************************************************
ok: [k8s-ubuntu-0]
ok: [k8s-ubuntu-1]
ok: [k8s-ubuntu-2]

TASK [ansible-role-kubernetes : Add K8s Repository] *****************************************************************************************************************
ok: [k8s-ubuntu-1]
ok: [k8s-ubuntu-2]
ok: [k8s-ubuntu-0]

TASK [ansible-role-kubernetes : Add Docker GPG apt Key] *************************************************************************************************************
ok: [k8s-ubuntu-1]
ok: [k8s-ubuntu-0]
ok: [k8s-ubuntu-2]

TASK [ansible-role-kubernetes : shell] ******************************************************************************************************************************
changed: [k8s-ubuntu-1]
changed: [k8s-ubuntu-2]
changed: [k8s-ubuntu-0]

TASK [ansible-role-kubernetes : Add Docker Repository] **************************************************************************************************************
ok: [k8s-ubuntu-0]
ok: [k8s-ubuntu-1]
ok: [k8s-ubuntu-2]

TASK [ansible-role-kubernetes : setup] ******************************************************************************************************************************
ok: [k8s-ubuntu-1]
ok: [k8s-ubuntu-0]
ok: [k8s-ubuntu-2]

TASK [ansible-role-kubernetes : include_tasks] **********************************************************************************************************************
included: /home/lorenzo/workspaces-local/ansible-role-kubernetes/tasks/preflight.yml for k8s-ubuntu-0, k8s-ubuntu-1, k8s-ubuntu-2

TASK [ansible-role-kubernetes : disable ufw] ************************************************************************************************************************
ok: [k8s-ubuntu-2]
ok: [k8s-ubuntu-0]
ok: [k8s-ubuntu-1]

TASK [ansible-role-kubernetes : Install iptables-legacy] ************************************************************************************************************
skipping: [k8s-ubuntu-0]
skipping: [k8s-ubuntu-1]
skipping: [k8s-ubuntu-2]

TASK [ansible-role-kubernetes : Remove zram-generator-defaults] *****************************************************************************************************
skipping: [k8s-ubuntu-0]
skipping: [k8s-ubuntu-1]
skipping: [k8s-ubuntu-2]

TASK [ansible-role-kubernetes : disable firewalld] ******************************************************************************************************************
skipping: [k8s-ubuntu-0]
skipping: [k8s-ubuntu-1]
skipping: [k8s-ubuntu-2]

TASK [ansible-role-kubernetes : Put SELinux in permissive mode, logging actions that would be blocked.] *************************************************************
skipping: [k8s-ubuntu-0]
skipping: [k8s-ubuntu-1]
skipping: [k8s-ubuntu-2]

TASK [ansible-role-kubernetes : Disable SELinux] ********************************************************************************************************************
skipping: [k8s-ubuntu-0]
skipping: [k8s-ubuntu-1]
skipping: [k8s-ubuntu-2]

TASK [ansible-role-kubernetes : Install openssl] ********************************************************************************************************************
ok: [k8s-ubuntu-2]
ok: [k8s-ubuntu-1]
ok: [k8s-ubuntu-0]

TASK [ansible-role-kubernetes : load overlay kernel module] *********************************************************************************************************
changed: [k8s-ubuntu-1]
changed: [k8s-ubuntu-0]
changed: [k8s-ubuntu-2]

TASK [ansible-role-kubernetes : load br_netfilter kernel module] ****************************************************************************************************
changed: [k8s-ubuntu-1]
changed: [k8s-ubuntu-0]
changed: [k8s-ubuntu-2]

[...]
[...]
[...]

TASK [ansible-role-kubernetes : Add KUBELET_ROOT_DIR env var] *******************************************************************************************************
skipping: [k8s-ubuntu-3]

TASK [ansible-role-kubernetes : Add KUBELET_ROOT_DIR env var, set value] ********************************************************************************************
skipping: [k8s-ubuntu-3]

TASK [ansible-role-kubernetes : Install longhorn] *******************************************************************************************************************
skipping: [k8s-ubuntu-3]

TASK [ansible-role-kubernetes : Install longhorn storageclass] ******************************************************************************************************
skipping: [k8s-ubuntu-3]

TASK [ansible-role-kubernetes : include_tasks] **********************************************************************************************************************
included: /home/lorenzo/workspaces-local/ansible-role-kubernetes/tasks/install_nginx_ingress.yml for k8s-ubuntu-3, k8s-ubuntu-4, k8s-ubuntu-5

TASK [ansible-role-kubernetes : Check if ingress-nginx is installed] ************************************************************************************************
changed: [k8s-ubuntu-3 -> k8s-ubuntu-0(192.168.25.110)]

TASK [ansible-role-kubernetes : Install ingress-nginx] **************************************************************************************************************
skipping: [k8s-ubuntu-3]

TASK [ansible-role-kubernetes : render nginx_ingress_config.yml] ****************************************************************************************************
skipping: [k8s-ubuntu-3]

TASK [ansible-role-kubernetes : Apply nginx ingress config] *********************************************************************************************************
skipping: [k8s-ubuntu-3]

PLAY RECAP **********************************************************************************************************************************************************
k8s-ubuntu-0               : ok=78   changed=24   unreachable=0    failed=0    skipped=25   rescued=0    ignored=3   
k8s-ubuntu-1               : ok=52   changed=12   unreachable=0    failed=0    skipped=30   rescued=0    ignored=1   
k8s-ubuntu-2               : ok=52   changed=12   unreachable=0    failed=0    skipped=30   rescued=0    ignored=1
k8s-ubuntu-3               : ok=58   changed=30   unreachable=0    failed=0    skipped=35   rescued=0    ignored=1   
k8s-ubuntu-4               : ok=52   changed=28   unreachable=0    failed=0    skipped=27   rescued=0    ignored=1   
k8s-ubuntu-5               : ok=52   changed=28   unreachable=0    failed=0    skipped=27   rescued=0    ignored=1   
```

Now we have a Kubernetes cluster deployed in high available mode, we can check the status of the cluster:

```
root@k8s-ubuntu-0:~# kubectl get nodes
NAME           STATUS   ROLES           AGE    VERSION
k8s-ubuntu-0   Ready    control-plane   139m   v1.24.3
k8s-ubuntu-1   Ready    control-plane   136m   v1.24.3
k8s-ubuntu-2   Ready    control-plane   136m   v1.24.3
k8s-ubuntu-3   Ready    <none>          117m   v1.24.3
k8s-ubuntu-4   Ready    <none>          117m   v1.24.3
k8s-ubuntu-5   Ready    <none>          117m   v1.24.3
```

Check the pods status:

```
root@k8s-ubuntu-0:~# kubectl get pods --all-namespaces
NAMESPACE         NAME                                           READY   STATUS      RESTARTS       AGE
ingress-nginx     ingress-nginx-admission-create-tsc8p           0/1     Completed   0              135m
ingress-nginx     ingress-nginx-admission-patch-48tpn            0/1     Completed   0              135m
ingress-nginx     ingress-nginx-controller-6dc865cd86-kfq88      1/1     Running     0              135m
kube-flannel      kube-flannel-ds-fm4s6                          1/1     Running     0              117m
kube-flannel      kube-flannel-ds-hhvxx                          1/1     Running     0              117m
kube-flannel      kube-flannel-ds-ngdtc                          1/1     Running     0              117m
kube-flannel      kube-flannel-ds-q5ncb                          1/1     Running     0              136m
kube-flannel      kube-flannel-ds-vq4kk                          1/1     Running     0              139m
kube-flannel      kube-flannel-ds-zshpf                          1/1     Running     0              137m
kube-system       coredns-6d4b75cb6d-8dh9h                       1/1     Running     0              139m
kube-system       coredns-6d4b75cb6d-xq98k                       1/1     Running     0              139m
kube-system       etcd-k8s-ubuntu-0                              1/1     Running     0              139m
kube-system       etcd-k8s-ubuntu-1                              1/1     Running     0              136m
kube-system       etcd-k8s-ubuntu-2                              1/1     Running     0              136m
kube-system       kube-apiserver-k8s-ubuntu-0                    1/1     Running     0              139m
kube-system       kube-apiserver-k8s-ubuntu-1                    1/1     Running     0              135m
kube-system       kube-apiserver-k8s-ubuntu-2                    1/1     Running     0              136m
kube-system       kube-controller-manager-k8s-ubuntu-0           1/1     Running     0              139m
kube-system       kube-controller-manager-k8s-ubuntu-1           1/1     Running     0              136m
kube-system       kube-controller-manager-k8s-ubuntu-2           1/1     Running     0              135m
kube-system       kube-proxy-59jqx                               1/1     Running     0              136m
kube-system       kube-proxy-8mjwr                               1/1     Running     0              139m
kube-system       kube-proxy-8nhbw                               1/1     Running     0              117m
kube-system       kube-proxy-j2rrx                               1/1     Running     0              117m
kube-system       kube-proxy-qwd5r                               1/1     Running     0              117m
kube-system       kube-proxy-vcs7g                               1/1     Running     0              137m
kube-system       kube-scheduler-k8s-ubuntu-0                    1/1     Running     0              139m
kube-system       kube-scheduler-k8s-ubuntu-1                    1/1     Running     0              136m
kube-system       kube-scheduler-k8s-ubuntu-2                    1/1     Running     0              135m
kube-system       kube-vip-k8s-ubuntu-0                          1/1     Running     1 (136m ago)   139m
kube-system       kube-vip-k8s-ubuntu-1                          1/1     Running     0              136m
kube-system       kube-vip-k8s-ubuntu-2                          1/1     Running     0              136m
longhorn-system   csi-attacher-dcb85d774-jrggr                   1/1     Running     0              114m
longhorn-system   csi-attacher-dcb85d774-slhqt                   1/1     Running     0              114m
longhorn-system   csi-attacher-dcb85d774-xcbxn                   1/1     Running     0              114m
longhorn-system   csi-provisioner-5d8dd96b57-74x6h               1/1     Running     0              114m
longhorn-system   csi-provisioner-5d8dd96b57-kdzdf               1/1     Running     0              114m
longhorn-system   csi-provisioner-5d8dd96b57-xmpjf               1/1     Running     0              114m
longhorn-system   csi-resizer-7c5bb5fd65-4262v                   1/1     Running     0              114m
longhorn-system   csi-resizer-7c5bb5fd65-mfjgv                   1/1     Running     0              114m
longhorn-system   csi-resizer-7c5bb5fd65-qw944                   1/1     Running     0              114m
longhorn-system   csi-snapshotter-5586bc7c79-bs2xn               1/1     Running     0              114m
longhorn-system   csi-snapshotter-5586bc7c79-d927b               1/1     Running     0              114m
longhorn-system   csi-snapshotter-5586bc7c79-v99t6               1/1     Running     0              114m
longhorn-system   engine-image-ei-766a591b-hrs6g                 1/1     Running     0              114m
longhorn-system   engine-image-ei-766a591b-n9fsn                 1/1     Running     0              114m
longhorn-system   engine-image-ei-766a591b-vxhbb                 1/1     Running     0              114m
longhorn-system   instance-manager-e-3dba6914                    1/1     Running     0              114m
longhorn-system   instance-manager-e-7bd8b1ff                    1/1     Running     0              114m
longhorn-system   instance-manager-e-aca0fdc4                    1/1     Running     0              114m
longhorn-system   instance-manager-r-244c040c                    1/1     Running     0              114m
longhorn-system   instance-manager-r-39bd81b1                    1/1     Running     0              114m
longhorn-system   instance-manager-r-3b7f12b1                    1/1     Running     0              114m
longhorn-system   longhorn-admission-webhook-858d86b96b-j5rcv    1/1     Running     0              135m
longhorn-system   longhorn-admission-webhook-858d86b96b-lphkq    1/1     Running     0              135m
longhorn-system   longhorn-conversion-webhook-576b5c45c7-4p55x   1/1     Running     0              135m
longhorn-system   longhorn-conversion-webhook-576b5c45c7-lq686   1/1     Running     0              135m
longhorn-system   longhorn-csi-plugin-f7zmn                      2/2     Running     0              114m
longhorn-system   longhorn-csi-plugin-hs58p                      2/2     Running     0              114m
longhorn-system   longhorn-csi-plugin-wfpfs                      2/2     Running     0              114m
longhorn-system   longhorn-driver-deployer-96cf98c98-7hzft       1/1     Running     0              135m
longhorn-system   longhorn-manager-92xws                         1/1     Running     0              116m
longhorn-system   longhorn-manager-b6knm                         1/1     Running     0              116m
longhorn-system   longhorn-manager-tg2zc                         1/1     Running     0              116m
longhorn-system   longhorn-ui-86b56b95c8-ctbvf                   1/1     Running     0              135m
```

we can see, longhorn, nginx ingress and all the kube-system pods.

We can also inspect the service of the nginx ingress controller:

```
root@k8s-ubuntu-0:~# kubectl get svc -n ingress-nginx
NAME                                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller                NodePort    10.111.203.177   <none>        80:30080/TCP,443:30443/TCP   136m
ingress-nginx-controller-admission      ClusterIP   10.105.11.11     <none>        443/TCP                      136m
```

we can see the nginx ingress controller listening port, in this case the http port is 30080 and  the https port is 30443. From an external machine we can test the ingress controller:

```
lorenzo@mint-virtual:~$ curl -v http://192.168.25.110:30080
*   Trying 192.168.25.110:30080...
* TCP_NODELAY set
* Connected to 192.168.25.110 (192.168.25.110) port 30080 (#0)
> GET / HTTP/1.1
> Host: 192.168.25.110:30080
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< Date: Wed, 17 Aug 2022 12:26:17 GMT
< Content-Type: text/html
< Content-Length: 146
< Connection: keep-alive
< 
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
* Connection #0 to host 192.168.25.110 left intact
```