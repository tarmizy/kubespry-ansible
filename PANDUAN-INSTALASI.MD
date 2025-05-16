# Panduan Lengkap: Instalasi Kubernetes dengan Kubespray dan Ansible di AlmaLinux

Dokumen ini merangkum langkah-langkah untuk menyiapkan cluster Kubernetes menggunakan Kubespray dan Ansible pada sistem operasi AlmaLinux.

## Daftar Isi
- [Persiapan Awal](#persiapan-awal)
- [Membuat User Ansible](#membuat-user-ansible)
- [Menyiapkan Kubespray](#menyiapkan-kubespray)
- [Konfigurasi Inventory](#konfigurasi-inventory)
- [Konfigurasi untuk Cluster Multi-Node](#konfigurasi-untuk-cluster-multi-node)
- [Persiapan Worker Node](#persiapan-worker-node)
- [Troubleshooting Umum](#troubleshooting-umum)
  - [Masalah SSH](#masalah-ssh)
  - [Grup Inventori Kosong](#grup-inventori-kosong)
  - [Masalah Konektivitas Node Worker](#masalah-konektivitas-node-worker)
- [Verifikasi Instalasi](#verifikasi-instalasi)

## Persiapan Awal

Update sistem dan instal paket yang diperlukan:

```bash
# Update sistem
sudo dnf update -y

# Instal paket yang diperlukan
sudo dnf install -y git python3 python3-pip sshpass epel-release
sudo dnf install -y ansible
```

## Membuat User Ansible

Buat user khusus untuk menjalankan Ansible:

```bash
# Buat user baru
sudo useradd -m ansible

# Set password untuk user ansible
sudo passwd ansible

# Berikan hak sudo tanpa password
echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible

# Pastikan file memiliki izin yang benar
sudo chmod 0440 /etc/sudoers.d/ansible
```

Login sebagai user ansible:

```bash
su - ansible
```

Konfigurasi SSH key:

```bash
# Generate SSH key
ssh-keygen -t rsa -b 4096 -N "" -f ~/.ssh/id_rsa

# Tampilkan public key
cat ~/.ssh/id_rsa.pub
```

## Menyiapkan Kubespray

Clone repository Kubespray dan instal dependensi:

```bash
# Buat direktori untuk workspace
mkdir ~/kubespray-setup
cd ~/kubespray-setup

# Clone kubespray repository
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray

# Install requirements
pip3 install --user -r requirements.txt
```

## Konfigurasi Inventory

Salin dan sesuaikan contoh inventory:

```bash
# Salin contoh inventory
cp -rfp inventory/sample inventory/mycluster

# Edit inventory
vi inventory/mycluster/inventory.ini
```

Untuk single-node cluster, gunakan konfigurasi berikut:

```ini
[all]
node1 ansible_host=127.0.0.1 ip=192.168.0.126 ansible_connection=local

[kube_control_plane]
node1

[etcd]
node1

[kube_node]
node1

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```

## Konfigurasi untuk Cluster Multi-Node

Untuk cluster dengan dua node atau lebih, konfigurasikan inventory seperti:

```ini
[all]
master ansible_host=127.0.0.1 ip=192.168.0.126 ansible_connection=local
worker1 ansible_host=192.168.0.121 ip=192.168.0.121

[kube_control_plane]
master

[etcd]
master

[kube_node]
worker1
master

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```

## Persiapan Worker Node

Pada worker node, jalankan langkah-langkah persiapan yang sama:

1. Buat user ansible:

```bash
# Buat user baru
sudo useradd -m ansible

# Set password untuk user ansible
sudo passwd ansible

# Berikan hak sudo tanpa password
echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible

# Pastikan file memiliki izin yang benar
sudo chmod 0440 /etc/sudoers.d/ansible
```

2. Konfigurasi SSH server:

```bash
sudo vi /etc/ssh/sshd_config
```

Pastikan konfigurasi berikut:
```
PasswordAuthentication yes
```

Restart SSH server:
```bash
sudo systemctl restart sshd
```

3. Dari node master, copy SSH key ke worker node:

```bash
# Login sebagai user ansible di node master
su - ansible

# Copy SSH key ke worker node
ssh-copy-id ansible@192.168.0.121

# Verifikasi koneksi tanpa password
ssh ansible@192.168.0.121
```

## Deployment Kubernetes Cluster

Jalankan playbook untuk menerapkan cluster:

```bash
cd ~/kubespray-setup/kubespray
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b
```

Untuk menjalankan hanya pada satu node tertentu:

```bash
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b --limit=worker1
```

## Troubleshooting Umum

### Masalah SSH

Jika mengalami masalah seperti:
```
fatal: [node1]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).", "unreachable": true}
```

Periksa:

1. Pastikan password authentication aktif di SSH server
2. Copy SSH key dengan benar
3. Untuk koneksi lokal, gunakan `ansible_connection=local`

### Grup Inventori Kosong

Jika muncul error:
```
failed: [node1] (item=kube_node) => { "assertion": "groups.get( item )", "changed": false, "evaluated_to": false, "item": "kube_node", "msg": "Assertion failed" }
```

Pastikan node ditetapkan ke semua grup yang diperlukan dalam inventory:
- `kube_control_plane`
- `etcd`
- `kube_node`

### Masalah Konektivitas Node Worker

Jika mengalami error saat worker mencoba bergabung dengan cluster:
```
error execution phase preflight: couldn't validate the identity of the API Server: dial tcp 192.168.0.126:6443: connect: no route to host
```

Langkah-langkah troubleshooting:

1. Periksa firewall di master node:
   ```bash
   # Buka port yang diperlukan
   sudo firewall-cmd --permanent --add-port=6443/tcp
   sudo firewall-cmd --permanent --add-port=10250/tcp
   sudo firewall-cmd --permanent --add-port=2379-2380/tcp
   sudo firewall-cmd --permanent --add-port=8472/udp
   sudo firewall-cmd --permanent --add-port=30000-32767/tcp
   sudo firewall-cmd --reload
   ```

2. Nonaktifkan SELinux sementara untuk pengujian:
   ```bash
   sudo setenforce 0
   ```

3. Verifikasi konektivitas dari worker ke master:
   ```bash
   # Test koneksi ke API server port
   telnet 192.168.0.126 6443
   ```

4. Pastikan API server mendengarkan pada alamat yang dapat diakses:
   ```bash
   sudo netstat -tulpn | grep 6443
   
   # Periksa log API server
   sudo journalctl -u kubelet
   sudo kubectl -n kube-system logs kube-apiserver-master
   
   # Jika perlu, edit konfigurasi API server untuk mendengarkan pada semua interface
   sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml
   # Pastikan --bind-address diatur ke 0.0.0.0 atau IP yang dapat diakses worker
   ```

## Verifikasi Instalasi

Setelah instalasi selesai, login ke node master dan verifikasi cluster:

```bash
# Cek node yang bergabung dalam cluster
sudo kubectl get nodes

# Cek status pod pada semua namespace
sudo kubectl get pods --all-namespaces

# Konfigurasi akses kubectl untuk user ansible
sudo mkdir -p /home/ansible/.kube
sudo cp /etc/kubernetes/admin.conf /home/ansible/.kube/config
sudo chown ansible:ansible /home/ansible/.kube/config
```

## Tips Tambahan

1. Untuk reset cluster atau node:
   ```bash
   ansible-playbook -i inventory/mycluster/inventory.ini reset.yml -b
   ```

2. Untuk memperbaiki satu node tertentu:
   ```bash
   ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b --limit=worker1
   ```

3. Untuk menjalankan playbook dengan informasi detail lebih lengkap:
   ```bash
   ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b -vvv
   ```

4. Pastikan semua node memiliki waktu yang tersinkronisasi dengan menggunakan NTP
