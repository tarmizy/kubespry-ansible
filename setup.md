    1  ssh-keygen -t rsa -b 4096 -N "" -f ~/.ssh/id_rsa
    2  mkdir ~/kubespray-setup
    3  cd ~/kubespray-setup
    4  git clone -b release-2.22 https://github.com/kubernetes-sigs/kubespray.git
    5  cd kubespray/
    6  pip3 install --user -r requirements.txt
    7  cp -rfp inventory/sample inventory/mycluster
    8  vim inventory/mycluster/inventory.ini
    9  vi inventory/mycluster/inventory.ini 
   10  vi inventory/mycluster/group_vars/all/all.yml
   11  cat inventory/mycluster/inventory.ini
   12  ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml --user=ansible --become --become-user=root cluster.yml
   13  ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b
   14  vi inventory/mycluster/inventory.ini 
   15  ssh-copy-id ansible@192.168.0.126
   16  sudo vi /etc/ssh/sshd_config
   17  sudo systemctl restart sshd
   18  chmod 600 ~/.ssh/id_rsa
   19  chmod 644 ~/.ssh/id_rsa.pub
   20  ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b -vvv
   21  vi inventory/mycluster/inventory.ini 
   22  hostname
   23  vi inventory/mycluster/inventory.ini 
   24  ssh-copy-id ansible@192.168.0.121
   25  ansible -i inventory/mycluster/inventory.ini all -m ping
   26  ansible -i inventory/mycluster/inventory.ini all -m shell -a "sudo whoami" -b
   27  ansible -i inventory/mycluster/inventory.ini all -m raw -a "which python3"
   28  ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b
   29  kubectl get nodes
   30  vi inventory/mycluster/inventory.ini 
   31  vi inventory/mycluster/group_vars/all/all.yml
   32  vi inventory/mycluster/inventory.ini 
   33  sudo kubectl get nodes
   34  kubectl get nodes
   35  sudo systemctl status kubelet
   36  ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b --limit=worker1
   37  vi /etc/kubernetes/manifests/kube-apiserver.yaml
   38  sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml
   39  ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b --limit=worker1
   40  sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml
   41  sudo systemctl status firewalld
   42  sudo systemctl stop firewalld
   43  netstat -pltn
   44  sudo kubectl -n kube-system logs kube-apiserver-master
   45  ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b
   46  sudo su
   47  sudo kubectl get nodes
   48  cat  /etc/kubernetes/admin.conf
   49  sudo cat  /etc/kubernetes/admin.conf
   50     curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   51  sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   52  chmod +x kubectl
   53  mkdir -p ~/.local/bin
   54  mv ./kubectl ~/.local/bin/kubectl
   55  kubectl get nodes
   56  sudo mkdir -p /home/ansible/.kube
   57  sudo cp /etc/kubernetes/admin.conf /home/ansible/.kube/config
   58  sudo chown ansible:ansible /home/ansible/.kube/config
   59  kubectl get nodes
   60  cd ..
   61  history > setup.md
