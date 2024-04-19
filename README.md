# Preflight

## `hosts.yaml` Configuration
Sesuaikan file `hosts.yaml` seperti contoh berikut
```
all:
  hosts:
    node1:
      ansible_host: 10.16.0.11
      ansible_name: master01
...
```

Notes:
- `ansible_host`: Reachable IP address
- `ansible_name`: Short/custom hostname

Contoh `/etc/hosts` yang akan jadi apabila ansible dijalankan
```
...
# BEGIN ANSIBLE MANAGED BLOCK
10.16.0.11 jh-k8s-master001 master01
10.16.0.12 jh-k8s-master001 master01
10.16.0.13 jh-k8s-master001 master01
10.16.0.14 jh-k8s-worker001 worker01
10.16.0.15 jh-k8s-worker002 worker02
10.16.0.16 jh-k8s-worker003 worker03
10.16.0.17 jh-k8s-worker004 worker04
# END ANSIBLE MANAGED BLOCK
```

Define host yang akan menjadi control plane ataupun worker
```
all:
...
  children:
    control_plane:
      hosts:
        node1:
        node2:
        node3:
    workers:
      hosts:
        node4:
        node5:
        node6:
        node7:
        node8:
```

## `variables.yaml` Configuration

### addons

Saat ini hanya support satu addons yaitu VRRP yang digunakan sebagai virtual IP address control plane loadbalancer yang menggunakan IPVS. Berikut contoh konfigurasinya
```yaml
addons:
  vrrp:
    enabled: true
    address: 10.16.0.10
    port: 6443
    interface: ens3
```

### containerd

Variable ini digunakan untuk mengganti root directory containerd yang biasanya digunakan untuk menempatkan root directory containerd ke partisi yang bukan untuk root filesystem
```yaml
containerd:
  directory: /mnt/containerd
```

### etc/hosts

Digunakan untuk menambahkan record sebuah host
```yaml
etc:
  hosts:
  - 10.16.0.10 kubeapi.beruanglaut.cloud
  - 10.0.0.1 proxy.beruanglaut.cloud
```

### kubernetes/apiserver

Variable ini dapat digunakan untuk mengganti default port Kube API server dan menambahkan entry sans certificate. Contoh konfigurasinya:
```yaml
kubernetes:
  apiserver:
    port: 8443
    sans:
    - kubeapi.beruanglaut.cloud
    - k8api.beruanglaut.cloud
```

### kubernetes/cluster

Digunakan untuk mengubah nama klaster, domain name dan DNS server Kubernetes
```yaml
kubernetes:
  cluster:
    name: beruanglaut
    domain: cluster.local
    dns: 169.254.20.10
```

### kubernetes/kubelet/container

Dapat mengkonfigurasi jumlah maksimal log size dan log file
```yaml
kubernetes:
  kubelet:
    container:
      logMaxSize: 25Mi
      logMaxFile: 2
```

### kubernetes/kubelet/eviction

Dapat mengkonfigurasi memory yang di-*reserve* untuk hard eviction
```yaml
kubernetes:
  kubelet:
    eviction:
      hard:
        memoryAvailable: 500Mi
```

> Note: Saat ini hanya support untuk hard eviction

### kubernetes/kubelet/reserved

Digunakan untuk mengalokasikan cpu dan memory ke kubelet dan container runtime (containerd)
```yaml
kubernetes:
  kubelet:
    reserved:
      enabled: true
      cgroup: runtime.slice
      cpu: 1000m
      memory: 1Gi
```

### kubernetes/proxy

Disable atau enable kube-proxy (default: true)
```yaml
kubernetes:
  proxy:
    enabled: false
```

### kubernetes/subnet

Mengatur subnet pod dan service kubernetes agar tidak collision dengan hostnya
```yaml
kubernetes:
  subnets:
    pod: 10.244.0.0/16
    service: 10.96.0.0/12
```

### kubernetes/system/reserved

Digunakan untuk mengalokasikan cpu dan memory ke sistem operasi
```yaml
kubernetes:
  system:
    reserved:
      enabled: true
      cgroup: system.slice
      cpu: 1000m
      memory: 1Gi
```

### kubernetes/version

Konfigurasi wajib untuk menentukan versi Kubernetes yang ingin digunakan (default: 1.26.9)
```yaml
kubernetes:
  version: 1.26.9
```

### proxy

Digunakan apabila environment tidak langsung mendapat akses Internet. Contoh konfigurasinya:
```
proxy:
  http_proxy: http://proxy.beruanglaut.cloud:8118
  https_proxy: http://proxy.beruanglaut.cloud:8118
  no_proxy: local,localhost,beruanglaut.cloud,127.0.0.1,10.96.0.1,10.96.0.10,169.254.20.10
```

### timesync

Variable yang berguna untuk mengkonfigurasi NTP sebagai salah satu requirement menjalankan Kubernetes. Default konfigurasinya sebagai berikut:
```
timesync:
  server: ntp.ubuntu.com
  zone: Asia/Jakarta
```

# HowTo's Deploy

>Notes: Sebelum menjalankan ansible pastikan user yang digunakan sudah bisa memakai sudo tanpa password

Test koneksi menggunakan
```
ansible -m ping all -u YOUR_USERNAME
```

Jalankan ansible
```
ansible-playbook kubernetes.yaml -u YOUR_USERNAME
```
