# Install kubeadm on ubuntu 20.4

1. <https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/>
1. <https://faun.pub/how-to-setup-kubernetes-cluster-with-kubeadm-on-ubuntu-20-04-ee197ac7a273>
1. <https://docs.oracle.com/en/operating-systems/olcne/1.1/start/netfilter.html>
1. <https://docs.oracle.com/en/operating-systems/olcne/1.1/start/network-bridge.html>
1. <https://www.vultr.com/docs/deploy-a-kubernetes-cluster-on-ubuntu-20-04/>

## Starting install

### Config Ubuntu Requirement

1. sudo apt -y install curl apt-transport-https
1. curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
1. echo "deb <https://apt.kubernetes.io/> kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
1. vi /etc/sysctl.d/99-kubernetes-cri.conf

   ```bash
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   net.ipv4.ip_forward = 1
   ```

1. swapoff -a
1. vi /etc/fstab

    ```bash
    # /etc/fstab: static file system information.
    #
    # Use 'blkid' to print the universally unique identifier for a
    # device; this may be used with UUID= as a more robust way to name devices
    # that works even if disks are added and removed. See fstab(5).
    #
    # <file system> <mount point>   <type>  <options>       <dump>  <pass>
    # / was on /dev/sda2 during curtin installation
    /dev/disk/by-uuid/78240eb0-0a32-4933-a064-dee5c0e31683 / ext4 defaults 0 0
    #/swap.img    none    swap    sw    0    0
    ```

1. vi /etc/security/limits.conf

   ```bash
   * soft nproc 65535
   * hard nproc 65535
   * soft nofile 65535
   * hard nofile 65535
   linuxhint soft nproc 100000 # user linuxhint
   linuxhint hard nproc 100000 # user linuxhint
   linuxhint soft nofile 100000 # user linuxhint
   linuxhint hard nofile 100000 # user linuxhint
   ```

1. reboot

### Install Container Runtime

1. <https://kubernetes.io/docs/setup/production-environment/container-runtimes/>
1. <https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-o>
1. <https://github.com/cri-o/cri-o#compatibility-matrix-cri-o--kubernetes>
1. vi /etc/modules-load.d/br_netfilter.conf

   ```bash
   br_netfilter
   ```

1. vi /etc/modules-load.d/overlay.conf

    ```bash
    overlay
    ```

1. reboot
1. Set CRI-O: export VERSION=1.23
1. Set Ubuntu: export OS=xUbuntu_20.04
1. echo "deb <https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/> /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
1. echo "deb <http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/>$VERSION/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
1. sudo curl -L <https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o>:$VERSION/$OS/Release.key | sudo apt-key add -
1. sudo curl -L <https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key> | sudo apt-key add -
1. apt update
1. apt install cri-o cri-o-runc -y
1. Find the path of container monitoring utility. `which conmon`
1. update crio.conf `vi /etc/crio/crio.conf`

    ```bash
    conmon = "/usr/bin/conmon"

    registries = [
        "quay.io",
        "docker.io",
        "gcr.io",
        "eu.gcr.io",
        "k8s.gcr.io"
    ]
    ```

1. systemctl daemon-reload
1. systemctl start crio
1. systemctl enable crio
1. systemctl status crio

### Install kubeadm, kubelet

1. Config kubelet using crio. `vi /etc/default/kubelet`
    ```bash
    KUBELET_EXTRA_ARGS= --container-runtime=remote --cgroup-driver=systemd --container-runtime-endpoint='unix:///var/run/crio/crio.sock' --runtime-request-timeout=5m
    ```
1. sudo apt update
1. sudo apt -y install kubelet kubeadm
1. sudo apt-mark hold kubelet kubeadm
1. systemctl restart kubelet
1. systemctl enable kubelet
1. vi kubeadm-config.yaml
    ```yaml
    apiVersion: kubeadm.k8s.io/v1beta3
    kind: ClusterConfiguration
    kubernetesVersion: 1.23.4 #<-- Use the word stable for newest version
    controlPlaneEndpoint: "k8s-master:6443" #<-- Use the node alias not the IP
    networking:
      podSubnet: 192.168.0.0/16
    ```
