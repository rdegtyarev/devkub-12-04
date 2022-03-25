# Домашнее задание к занятию "12.4 Развертывание кластера на собственных серверах, лекция 2"
Новые проекты пошли стабильным потоком. Каждый проект требует себе несколько кластеров: под тесты и продуктив. Делать все руками — не вариант, поэтому стоит автоматизировать подготовку новых кластеров.

## Задание 1: Подготовить инвентарь kubespray

<details>

  <summary>Описание задачи</summary>  
Для экспериментов и валидации ваших решений вам нужно подготовить тестовую среду для работы с Kubernetes. Оптимальное решение — развернуть на рабочей машине Minikube.

Новые тестовые кластеры требуют типичных простых настроек. Нужно подготовить инвентарь и проверить его работу. Требования к инвентарю:
* подготовка работы кластера из 5 нод: 1 мастер и 4 рабочие ноды;
* в качестве CRI — containerd;
* запуск etcd производить на мастере.

</details>  
  
- Клонируем репозиторий
```shell script
git clone https://github.com/kubernetes-sigs/kubespray
```

- Создаем inventory из примера
```shell script
cp -rfp inventory/sample inventory/mycluster
```


- Билдим host.yaml
```bash
declare -a IPS=(10.128.0.9 10.128.0.3 10.128.0.23 10.128.0.36 10.128.0.28)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```  

- Проверяем результат и вносим правки
```yaml
all:
  hosts:
    cp1:
      ansible_host: 10.128.0.28
      ip: 10.128.0.28
      access_ip: 10.128.0.28
    node0:
      ansible_host: 10.128.0.9
      ip: 10.128.0.9
      access_ip: 10.128.0.9
    node1:
      ansible_host: 10.128.0.3
      ip: 10.128.0.3
      access_ip: 10.128.0.3
    node2:
      ansible_host: 10.128.0.23
      ip: 10.128.0.23
      access_ip: 10.128.0.23
    node3:
      ansible_host: 10.128.0.36
      ip: 10.128.0.36
      access_ip: 10.128.0.36
  children:
    kube_control_plane:
      hosts:
        cp1: # сontrol-plane
    kube_node:
      hosts: # рабочие ноды
        node0: 
        node1:
        node2:
        node3:
    etcd: # размещаем etcd на сontrol-plane
      hosts:
        cp1:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
```
  
- Проверяем что в group_vars/k8s_cluster/k8s-cluster.yml указан container_manager - containerd
```yaml
container_manager: containerd
```
  
- Запускаем плейбук:
```bash
 ansible-playbook -i inventory/mycluster/hosts.yaml cluster.yml -b -v
 ```

После установки проверяем:
```bash
kubectl get nodes -o wide
NAME    STATUS   ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
cp1     Ready    control-plane,master   27m   v1.23.5   10.128.0.28   <none>        Ubuntu 20.04.3 LTS   5.4.0-42-generic   containerd://1.6.1
node0   Ready    <none>                 26m   v1.23.5   10.128.0.9    <none>        Ubuntu 20.04.3 LTS   5.4.0-42-generic   containerd://1.6.1
node1   Ready    <none>                 26m   v1.23.5   10.128.0.3    <none>        Ubuntu 20.04.3 LTS   5.4.0-42-generic   containerd://1.6.1
node2   Ready    <none>                 26m   v1.23.5   10.128.0.23   <none>        Ubuntu 20.04.3 LTS   5.4.0-42-generic   containerd://1.6.1
node3   Ready    <none>                 26m   v1.23.5   10.128.0.36   <none>        Ubuntu 20.04.3 LTS   5.4.0-42-generic   containerd://1.6.1
```
---

## Задание 2 (*): подготовить и проверить инвентарь для кластера в AWS


<details>

  <summary>Описание задачи</summary> 

Часть новых проектов хотят запускать на мощностях AWS. Требования похожи:
* разворачивать 5 нод: 1 мастер и 4 рабочие ноды;
* работать должны на минимально допустимых EC2 — t3.small.

</details>

- Указанные выше действия выполнены на ресурсах Yandex Cloud. 
- Доступ в подсеть осуществляется через OpenVPN, поэтому используются локальные IP адреса нод.

---