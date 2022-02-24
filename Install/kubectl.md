# Install kubectl on ubuntu 20.04

1. <https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/>
1. sudo apt-get update
1. sudo apt-get install -y apt-transport-https ca-certificates curl
1. sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
1. echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
1. sudo apt-get update
1. sudo apt-get install -y kubectl
