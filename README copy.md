<h2 class="code-line" data-line-start=0 data-line-end=1 ><a id="____124_______2_0"></a>Домашнее задание к занятию “12.4 Развертывание кластера на собственных серверах, лекция 2”</h2>
<p class="has-line-data" data-line-start="2" data-line-end="3">Клонировал репозиторий</p>
<pre><code>https://github.com/kubernetes-sigs/kubespray
</code></pre>
<p class="has-line-data" data-line-start="6" data-line-end="7">Скопировал пример в свою директорию</p>
<pre><code>cp -rfp inventory/sample inventory/mycluster
</code></pre>
<p class="has-line-data" data-line-start="9" data-line-end="10">Создал в яндекс облаке виртуальные машины и зашел к ним по SSH</p>
<pre><code>+----------------------+-------+---------------+---------+---------------+-------------+
|          ID          | NAME  |    ZONE ID    | STATUS  |  EXTERNAL IP  | INTERNAL IP |
+----------------------+-------+---------------+---------+---------------+-------------+
| ef34omg3imf6vl3h4jfo | node2 | ru-central1-c | RUNNING | 51.250.44.83  | 10.130.0.24 |
| ef39vocp45cebisv7n8q | node1 | ru-central1-c | RUNNING | 51.250.36.33  | 10.130.0.35 |
| ef3jqb4h3s9glad47k37 | cp1   | ru-central1-c | RUNNING | 51.250.44.250 | 10.130.0.3  |
| ef3qqej5qa28dle1vfc0 | node3 | ru-central1-c | RUNNING | 51.250.42.109 | 10.130.0.14 |
+----------------------+-------+---------------+---------+---------------+-------------+
</code></pre>
<p class="has-line-data" data-line-start="20" data-line-end="21">Запустил билдер  и подготовил inventory/mycluster/hosts.yaml</p>
<pre><code>declare -a IPS=(51.250.44.250 51.250.36.33 51.250.44.83 51.250.42.109)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
</code></pre>
<p class="has-line-data" data-line-start="24" data-line-end="25">Внес изменения</p>
<pre><code>hosts.yaml
all:
  hosts:
    cp1:
      ansible_host: 51.250.44.250
      ansible_user: yc-user
    node1:
      ansible_host: 51.250.36.33
      ansible_user: yc-user
    node2:
      ansible_host: 51.250.44.83
      ansible_user: yc-user
    node3:
      ansible_host: 51.250.42.109
      ansible_user: yc-user  
  children:
    kube_control_plane:
      hosts:
        cp1:
    kube_node:
      hosts:
        node1:
        node2:
        node3:
    etcd:
      hosts:
        cp1:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}

all.yml
loadbalancer_apiserver:
  address: 51.250.40.49
  port: 6443

k8s-cluster.yml
## Container runtime
## docker for docker, crio for cri-o and containerd for containerd.
## Default: containerd
container_manager: containerd
</code></pre>
<p class="has-line-data" data-line-start="71" data-line-end="72">Запустил ansible-playbook</p>
<pre><code>ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml
</code></pre>
<p class="has-line-data" data-line-start="75" data-line-end="76">Скрипт работал 18 минут</p>
<pre><code>Friday 20 May 2022  14:57:28 +0300 (0:00:00.097)       0:17:52.527 ************
===============================================================================
download : download_container | Download image if required ----------------------------------------------------- 58.13s
network_plugin/calico : Wait for calico kubeconfig to be created ----------------------------------------------- 52.75s
kubernetes/control-plane : kubeadm | Initialize first master --------------------------------------------------- 46.71s
download : download_container | Download image if required ----------------------------------------------------- 39.09s
kubernetes/preinstall : Install packages requirements ---------------------------------------------------------- 37.75s
kubernetes/kubeadm : Join to cluster --------------------------------------------------------------------------- 32.71s
download : download_container | Download image if required ----------------------------------------------------- 29.11s
download : download_container | Download image if required ----------------------------------------------------- 26.00s
download : download_file | Validate mirrors -------------------------------------------------------------------- 25.09s
download : download_container | Download image if required ----------------------------------------------------- 22.50s
download : download_container | Download image if required ----------------------------------------------------- 18.16s
kubernetes-apps/ansible : Kubernetes Apps | Start Resources ---------------------------------------------------- 17.85s
download : download_container | Download image if required ----------------------------------------------------- 14.00s
kubernetes/preinstall : Preinstall | wait for the apiserver to be running -------------------------------------- 12.09s
download : download_file | Download item ----------------------------------------------------------------------- 10.38s
kubernetes/preinstall : Update package management cache (APT) -------------------------------------------------- 10.00s
kubernetes/node : install | Copy kubelet binary from download dir ----------------------------------------------- 9.62s
etcd : reload etcd ---------------------------------------------------------------------------------------------- 9.19s
download : download_container | Download image if required ------------------------------------------------------ 9.08s
kubernetes-apps/ansible : Kubernetes Apps | Lay Down CoreDNS templates ------------------------------------------ 8.51s
</code></pre>
<p class="has-line-data" data-line-start="100" data-line-end="101">Далее зашел на control_plane</p>
<pre><code>yc-user@cp1:~$ sudo cat /etc/kubernetes/admin.conf
</code></pre>
<p class="has-line-data" data-line-start="103" data-line-end="104">Скопировал содержимоев локальный конфиг подправил сервер балансировщик</p>
<pre><code>server: https://51.250.44.250:6443
</code></pre>
<p class="has-line-data" data-line-start="106" data-line-end="107">Получил список нод</p>
<pre><code>mike@HOMEDX79SR:~/kb124$ kubectl get nodes
NAME    STATUS   ROLES                  AGE     VERSION
cp1     Ready    control-plane,master   9m45s   v1.23.6
node1   Ready    &lt;none&gt;                 8m37s   v1.23.6
node2   Ready    &lt;none&gt;                 8m37s   v1.23.6
node3   Ready    &lt;none&gt;                 8m37s   v1.23.6
</code></pre>
<p class="has-line-data" data-line-start="115" data-line-end="116">Для примера задеплоил nginx:latest</p>
<pre><code>mike@HOMEDX79SR:~/kb124$ kubectl create deploy nginx --image=nginx:latest --replicas=2
ubectl get po -o widedeployment.apps/nginx created
mike@HOMEDX79SR:~/kb124$ kubectl get po -o wide
NAME                     READY   STATUS              RESTARTS   AGE   IP       NODE    NOMINATED NODE   READINESS GATES
nginx-7c658794b9-5hk69   0/1     ContainerCreating   0          4s    &lt;none&gt;   node1   &lt;none&gt;           &lt;none&gt;
nginx-7c658794b9-swft4   0/1     ContainerCreating   0          4s    &lt;none&gt;   node2   &lt;none&gt;           &lt;none&gt;
</code></pre>