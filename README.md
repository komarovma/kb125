<h2 class="code-line" data-line-start=0 data-line-end=1 ><a id="____125___CNI_0"></a>Домашнее задание по лекции “12.5. Сетевые решения CNI”</h2>
<p class="has-line-data" data-line-start="2" data-line-end="3">Клонировал репозиторий (там по умолчанию calico)</p>
<pre><code>https://github.com/kubernetes-sigs/kubespray
</code></pre>
<p class="has-line-data" data-line-start="5" data-line-end="6">Скопировал пример в свою директорию</p>
<pre><code>cp -rfp inventory/sample inventory/mycluster
</code></pre>
<p class="has-line-data" data-line-start="8" data-line-end="9">Создал в яндекс облаке виртуальные машины и зашел к ним по SSH</p>
<pre><code>PS Q:\Netologia\Yandex&gt; yc compute instance list
+----------------------+-------+---------------+---------+---------------+-------------+
|          ID          | NAME  |    ZONE ID    | STATUS  |  EXTERNAL IP  | INTERNAL IP |
+----------------------+-------+---------------+---------+---------------+-------------+
| ef36pvpv1qjlj66p3oan | node1 | ru-central1-c | RUNNING | 51.250.43.150 | 10.130.0.25 |
| ef39fndfnfvefq80b1h5 | cp1   | ru-central1-c | RUNNING | 51.250.32.89  | 10.130.0.10 |
| ef3lapekphocisf2pekc | node2 | ru-central1-c | RUNNING | 51.250.46.102 | 10.130.0.18 |
| ef3sokhnut2e0t02k43o | node3 | ru-central1-c | RUNNING | 51.250.38.179 | 10.130.0.29 |
+----------------------+-------+---------------+---------+---------------+-------------+
</code></pre>
<p class="has-line-data" data-line-start="20" data-line-end="21">Запустил билдер  и подготовил inventory/mycluster/hosts.yaml</p>
<pre><code>declare -a IPS=(51.250.32.89 51.250.43.150 51.250.46.102 51.250.38.179)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
</code></pre>
<p class="has-line-data" data-line-start="24" data-line-end="25">Внес изменения</p>
<pre><code>hosts.yaml
all:
  hosts:
    cp1:
      ansible_host: 51.250.32.89
      ansible_user: yc-user
    node1:
      ansible_host: 51.250.43.150
      ansible_user: yc-user
    node2:
      ansible_host: 51.250.46.102
      ansible_user: yc-user
    node3:
      ansible_host: 51.250.38.179
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
  address: 51.250.32.89
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
<pre><code>Friday 22 May 2022  10:57:28 +0300 (0:00:00.097)       0:17:52.527 ************
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
<p class="has-line-data" data-line-start="103" data-line-end="104">Скопировал содержимое в локальный конфиг подправил сервер балансировщик</p>
<pre><code>server: https://51.250.44.250:6443
</code></pre>
<p class="has-line-data" data-line-start="106" data-line-end="107">Список нод</p>
<pre><code>mike@HOMEDX79SR:~/kb125$ kubectl get node
NAME    STATUS   ROLES                  AGE     VERSION
cp1     Ready    control-plane,master   8m48s   v1.23.6
node1   Ready    &lt;none&gt;                 7m25s   v1.23.6
node2   Ready    &lt;none&gt;                 7m25s   v1.23.6
node3   Ready    &lt;none&gt;                 7m25s   v1.23.6
</code></pre>
<p class="has-line-data" data-line-start="115" data-line-end="116">Скопировал с доп материалов с курса (20-network-policy) на ходиться в репозитории</p>
<pre><code>https://github.com/aak74/kubernetes-for-beginners
</code></pre>
<p class="has-line-data" data-line-start="119" data-line-end="120">Деплою пример</p>
<pre><code>mike@HOMEDX79SR:~/kb125/20-network-policy$ kubectl apply -f ./templates/main/
deployment.apps/frontend created
service/frontend created
deployment.apps/backend created
service/backend created
deployment.apps/cache created
service/cache created
</code></pre>
<p class="has-line-data" data-line-start="129" data-line-end="130">Проверяю развертывание</p>
<pre><code>mike@HOMEDX79SR:~/kb125/20-network-policy$ kubectl get po
NAME                        READY   STATUS    RESTARTS   AGE
backend-f785447b9-lgwdh     1/1     Running   0          24s
cache-b4f65b647-vqtwd       1/1     Running   0          24s
frontend-8645d9cb9c-wpwk4   1/1     Running   0          24s
</code></pre>
<p class="has-line-data" data-line-start="137" data-line-end="138">Проверка доступности между подами</p>
<pre><code>mike@HOMEDX79SR:~/kb125/20-network-policy$ kubectl exec backend-f785447b9-lgwdh -- curl -s -m 1 frontend
bectl exec backend-f785447b9-lgwdh -- curl -s -m 1 cache
kubectl exec backend-f785447b9-lgwdh -- curl -s -m 1 backendPraqma Network MultiTool (with NGINX) - frontend-8645d9cb9c-wpwk4 - 10.233.90.1
mike@HOMEDX79SR:~/kb125/20-network-policy$ kubectl exec backend-f785447b9-lgwdh -- curl -s -m 1 cache
Praqma Network MultiTool (with NGINX) - cache-b4f65b647-vqtwd - 10.233.96.2
mike@HOMEDX79SR:~/kb125/20-network-policy$ kubectl exec backend-f785447b9-lgwdh -- curl -s -m 1 backend
Praqma Network MultiTool (with NGINX) - backend-f785447b9-lgwdh - 10.233.92.1
</code></pre>
<p class="has-line-data" data-line-start="147" data-line-end="148">Применение NetworkPolicy</p>
<pre><code>mike@HOMEDX79SR:~/kb125/20-network-policy$ kubectl apply -f ./templates/network-policy
networkpolicy.networking.k8s.io/default-deny-ingress created
networkpolicy.networking.k8s.io/frontend created
networkpolicy.networking.k8s.io/backend created
networkpolicy.networking.k8s.io/cache created
</code></pre>
<p class="has-line-data" data-line-start="155" data-line-end="156">Проверка наличия политик</p>
<pre><code>mike@HOMEDX79SR:~/kb125/20-network-policy$ kubectl get networkpolicies
NAME                   POD-SELECTOR   AGE
backend                app=backend    38s
cache                  app=cache      38s
default-deny-ingress   &lt;none&gt;         38s
frontend               app=frontend   38s
</code></pre>
<p class="has-line-data" data-line-start="164" data-line-end="165">Проверяем применение политик, согласно политик часть взаимодействий заблокировано</p>
<pre><code>Mike@HOMEDX79SR:~/kb125/20-network-policy$ kubectl exec backend-f785447b9-lgwdh -- curl -s -m 1 frontend
bectl exec backend-f785447b9-lgwdh -- curl -s -m 1 cache
kubectl exec backend-f785447b9-lgwdh -- curl -s -m 1 backendcommand terminated with exit code 28
mike@HOMEDX79SR:~/kb125/20-network-policy$ kubectl exec backend-f785447b9-lgwdh -- curl -s -m 1 cache
Praqma Network MultiTool (with NGINX) - cache-b4f65b647-vqtwd - 10.233.96.2
mike@HOMEDX79SR:~/kb125/20-network-policy$ kubectl exec backend-f785447b9-lgwdh -- curl -s -m 1 backend
command terminated with exit code 28
</code></pre>
<p class="has-line-data" data-line-start="174" data-line-end="175">Ставим утилиту calicoctl</p>
<pre><code>https://projectcalico.docs.tigera.io/maintenance/clis/calicoctl/install


mike@HOMEDX79SR:~$ ./calicoctl get ipPool --allow-version-mismatch
NAME           CIDR             SELECTOR
default-pool   10.233.64.0/18   all()

mike@HOMEDX79SR:~$ ./calicoctl get profile --allow-version-mismatch
NAME
projectcalico-default-allow
kns.default
kns.kube-node-lease
kns.kube-public
kns.kube-system
ksa.default.default
ksa.kube-node-lease.default</code></pre>