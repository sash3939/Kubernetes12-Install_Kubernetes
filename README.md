# Домашнее задание к занятию «Установка Kubernetes»

### Цель задания

Установить кластер K8s.

### Чеклист готовности к домашнему заданию

1. Развёрнутые ВМ с ОС Ubuntu 20.04-lts.

Необходимо установить 23 версию Ubuntu, а не ту, что в этой Нетологии предлагают. Совсем обленились и ничего не обновляют в лекциях и презентациях. Позорно.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция по установке kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/).
2. [Документация kubespray](https://kubespray.io/).

-----

### Задание 1. Установить кластер k8s с 1 master node

1. Подготовка работы кластера из 5 нод: 1 мастер и 4 рабочие ноды.
2. В качестве CRI — containerd.
3. Запуск etcd производить на мастере.
4. Способ установки выбрать самостоятельно.


Решение
Создаем в yc скриптом необходимое количество виртуальных машин

```bash
root@kuber:~# yc init
Welcome! This command will take you through the configuration process.
Please go to https://oauth.yandex.ru/authorize?response_type=token&client_id=1a6990aa636648e9b2ef855fa7bec2fb in order to obtain OAuth token.
 Please enter OAuth token: <token_here>
You have one cloud available: 'netology-cloud-s1a9s8h6a' (id = b1g0tptlsvgkbge2ul42). It is going to be used by default.
Please choose folder to use:
 [1] netology (id = b1gc36q9v49llnddjkvr)
 [2] Create a new folder
Please enter your numeric choice: 1
Your current folder has been set to 'netology' (id = b1gc36q9v49llnddjkvr).
Do you want to configure a default Compute zone? [Y/n] Y
Which zone do you want to use as a profile default?
 [1] ru-central1-a
 [2] ru-central1-b
 [3] ru-central1-d
 [4] Don't set default zone
Please enter your numeric choice: 1
Your profile default Compute zone has been set to 'ru-central1-a'.
root@kuber:~# 
root@kuber:~/Kubernetes12-Install_Kubernetes/vms_yc# yc vpc network create  --name net --labels my-label=netology --description "net yc"
id: enp0nfcfhb61hlntjqcb
folder_id: b1gc36q9v49llnddjkvr
created_at: "2024-12-04T13:17:01Z"
name: net
description: net yc
labels:
  my-label: netology
default_security_group_id: enpsfk9occv6169bdd8l

root@kuber:~/Kubernetes12-Install_Kubernetes/vms_yc# yc vpc subnet create  --name my-subnet --zone ru-central1-a --range 10.1.2.0/24 --network-name net --description "subnet yc"
id: e9be4aqs9si3tvcm6tfb
folder_id: b1gc36q9v49llnddjkvr
created_at: "2024-12-04T13:17:59Z"
name: my-subnet
description: subnet yc
network_id: enp0nfcfhb61hlntjqcb
zone_id: ru-central1-a
v4_cidr_blocks:
  - 10.1.2.0/24


```

```bash
root@kuber:~/Kubernetes12-Install_Kubernetes/vms_yc# bash create-vms.sh 
done (49s)
id: fhmum1096qohn8jjb7up
folder_id: b1gc36q9v49llnddjkvr
created_at: "2024-12-04T13:18:28Z"
name: masterk8s
zone_id: ru-central1-a
platform_id: standard-v2
resources:
  memory: "2147483648"
  cores: "2"
  core_fraction: "100"
status: RUNNING
metadata_options:
  gce_http_endpoint: ENABLED
  aws_v1_http_endpoint: ENABLED
  gce_http_token: ENABLED
  aws_v1_http_token: DISABLED
boot_disk:
  mode: READ_WRITE
  device_name: fhmge1pddjrhhassbdr1
  auto_delete: true
  disk_id: fhmge1pddjrhhassbdr1
network_interfaces:
  - index: "0"
    mac_address: d0:0d:1e:b0:40:93
    subnet_id: e9be4aqs9si3tvcm6tfb
    primary_v4_address:
      address: 10.1.2.9
      one_to_one_nat:
        address: 51.250.70.251
        ip_version: IPV4
serial_port_settings:
  ssh_authorization: OS_LOGIN
gpu_settings: {}
fqdn: masterk8s.ru-central1.internal
scheduling_policy:
  preemptible: true
network_settings:
  type: STANDARD
placement_policy: {}
hardware_generation:
  legacy_features:
    pci_topology: PCI_TOPOLOGY_V1

done (39s)
id: fhmo6evlmldc3chmpn03
folder_id: b1gc36q9v49llnddjkvr
created_at: "2024-12-04T13:19:21Z"
name: worker1
zone_id: ru-central1-a
platform_id: standard-v2
resources:
  memory: "2147483648"
  cores: "2"
  core_fraction: "100"
status: RUNNING
metadata_options:
  gce_http_endpoint: ENABLED
  aws_v1_http_endpoint: ENABLED
  gce_http_token: ENABLED
  aws_v1_http_token: DISABLED
boot_disk:
  mode: READ_WRITE
  device_name: fhm39kqgnp1lpb2t54gs
  auto_delete: true
  disk_id: fhm39kqgnp1lpb2t54gs
network_interfaces:
  - index: "0"
    mac_address: d0:0d:18:33:bf:5b
    subnet_id: e9be4aqs9si3tvcm6tfb
    primary_v4_address:
      address: 10.1.2.32
      one_to_one_nat:
        address: 89.169.142.23
        ip_version: IPV4
serial_port_settings:
  ssh_authorization: OS_LOGIN
gpu_settings: {}
fqdn: worker1.ru-central1.internal
scheduling_policy:
  preemptible: true
network_settings:
  type: STANDARD
placement_policy: {}
hardware_generation:
  legacy_features:
    pci_topology: PCI_TOPOLOGY_V1

done (39s)
id: fhm6r2k61um4r5lv0q1v
folder_id: b1gc36q9v49llnddjkvr
created_at: "2024-12-04T13:20:05Z"
name: worker2
zone_id: ru-central1-a
platform_id: standard-v2
resources:
  memory: "2147483648"
  cores: "2"
  core_fraction: "100"
status: RUNNING
metadata_options:
  gce_http_endpoint: ENABLED
  aws_v1_http_endpoint: ENABLED
  gce_http_token: ENABLED
  aws_v1_http_token: DISABLED
boot_disk:
  mode: READ_WRITE
  device_name: fhmiobm2j1thcaof29pn
  auto_delete: true
  disk_id: fhmiobm2j1thcaof29pn
network_interfaces:
  - index: "0"
    mac_address: d0:0d:6d:8a:86:0f
    subnet_id: e9be4aqs9si3tvcm6tfb
    primary_v4_address:
      address: 10.1.2.24
      one_to_one_nat:
        address: 89.169.151.247
        ip_version: IPV4
serial_port_settings:
  ssh_authorization: OS_LOGIN
gpu_settings: {}
fqdn: worker2.ru-central1.internal
scheduling_policy:
  preemptible: true
network_settings:
  type: STANDARD
placement_policy: {}
hardware_generation:
  legacy_features:
    pci_topology: PCI_TOPOLOGY_V1

done (33s)
id: fhmoa3jkdc4iupk2fmo6
folder_id: b1gc36q9v49llnddjkvr
created_at: "2024-12-04T13:20:47Z"
name: worker3
zone_id: ru-central1-a
platform_id: standard-v2
resources:
  memory: "2147483648"
  cores: "2"
  core_fraction: "100"
status: RUNNING
metadata_options:
  gce_http_endpoint: ENABLED
  aws_v1_http_endpoint: ENABLED
  gce_http_token: ENABLED
  aws_v1_http_token: DISABLED
boot_disk:
  mode: READ_WRITE
  device_name: fhmbudsjl78ingnn128i
  auto_delete: true
  disk_id: fhmbudsjl78ingnn128i
network_interfaces:
  - index: "0"
    mac_address: d0:0d:18:50:e7:46
    subnet_id: e9be4aqs9si3tvcm6tfb
    primary_v4_address:
      address: 10.1.2.17
      one_to_one_nat:
        address: 89.169.156.35
        ip_version: IPV4
serial_port_settings:
  ssh_authorization: OS_LOGIN
gpu_settings: {}
fqdn: worker3.ru-central1.internal
scheduling_policy:
  preemptible: true
network_settings:
  type: STANDARD
placement_policy: {}
hardware_generation:
  legacy_features:
    pci_topology: PCI_TOPOLOGY_V1

done (38s)
id: fhmj95c9jimde61c54sp
folder_id: b1gc36q9v49llnddjkvr
created_at: "2024-12-04T13:21:23Z"
name: worker4
zone_id: ru-central1-a
platform_id: standard-v2
resources:
  memory: "2147483648"
  cores: "2"
  core_fraction: "100"
status: RUNNING
metadata_options:
  gce_http_endpoint: ENABLED
  aws_v1_http_endpoint: ENABLED
  gce_http_token: ENABLED
  aws_v1_http_token: DISABLED
boot_disk:
  mode: READ_WRITE
  device_name: fhmttmiitberelbqurkf
  auto_delete: true
  disk_id: fhmttmiitberelbqurkf
network_interfaces:
  - index: "0"
    mac_address: d0:0d:13:49:58:99
    subnet_id: e9be4aqs9si3tvcm6tfb
    primary_v4_address:
      address: 10.1.2.10
      one_to_one_nat:
        address: 89.169.141.94
        ip_version: IPV4
serial_port_settings:
  ssh_authorization: OS_LOGIN
gpu_settings: {}
fqdn: worker4.ru-central1.internal
scheduling_policy:
  preemptible: true
network_settings:
  type: STANDARD
placement_policy: {}
hardware_generation:
  legacy_features:
    pci_topology: PCI_TOPOLOGY_V1


```

Прокидываем в мастер ssh ключ. Делаем мы это с той машины, с которой подключаемся к мастеру (мастер у нас в данном случае в ЯО как обычная ВМ на Ubuntu). Конечно заранее на хостовой машине ключ должен быть сгенерирован

scp /root/.ssh/id_ed25519 yc-user@89.169.148.181:.ssh/id_ed25519

## Вход на ВМ

ssh -l yc-user <ext ip> или ssh yc-user@<ext ip VM>

<img width="1119" alt="VM" src="https://github.com/user-attachments/assets/74001bb2-7460-4a40-8d2e-c4041a41a64e">

## Ставим компоненты на мастере
sudo apt update
sudo apt install git python3 python3-pip python3.12-venv -y
Еще нужно поставить это
sudo apt install python3.8-venv

## Делаем клон под пользователем, которому мы кстати скопировали ssh ключ, именно ему.

git clone https://github.com/kubernetes-incubator/kubespray.git
cd kubespray
git checkout release-2.26

<img width="330" alt="kubespray" src="https://github.com/user-attachments/assets/3afae01f-5847-48ef-b664-7ee180fc09ea">

## Создаем и активируем в виртуальную среду
python3 -m venv .venv
source .venv/bin/activate

<img width="317" alt="activate venv" src="https://github.com/user-attachments/assets/23d12d5f-85ee-4e7e-aca5-6af3484a8eaf">

## Обязательно переходим на другой релиз, на 2.26 (уже ранее перешли на эту ветку)
cd kubespray

## Установка зависимостей
python3 -m pip install -r requirements.txt

<img width="761" alt="requirements" src="https://github.com/user-attachments/assets/11198a4b-f37c-43c3-a550-0dd50b2544c3">

## Подготовим файл host.yaml

cp -rfp inventory/sample inventory/mycluster
declare -a IPS=(10.1.2.3 10.1.2.32 10.1.2.21 10.1.2.28 10.1.2.7)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

<img width="523" alt="for hosts" src="https://github.com/user-attachments/assets/78091eab-a574-4f24-97d5-36d4bc093900">

<img width="430" alt="hosts" src="https://github.com/user-attachments/assets/412721b8-cfa1-41cc-bc4a-a542ad222059">

<img width="562" alt="containerd" src="https://github.com/user-attachments/assets/99a45aa8-f838-401e-b50d-a15e0c9b7fe5">


## Добавим PRIVATE KEY на master чтобы ansible имел возможность подключаться к worker (ранее уже добавили)

<img width="321" alt="ed25519" src="https://github.com/user-attachments/assets/bc3820bf-d18e-4008-aa6b-6f1c0fca8072">


## Запуск playbook

ansible-playbook  -i inventory/mycluster/hosts.yaml cluster.yml -b -v &

<img width="733" alt="ansible after worked" src="https://github.com/user-attachments/assets/6dd9a001-9ee2-49c8-900b-1b643bf1fe26">

<img width="716" alt="check work" src="https://github.com/user-attachments/assets/68cc694b-75d0-4dd8-954a-774015107d22">

<img width="697" alt="Install nginx" src="https://github.com/user-attachments/assets/50b82c73-d7fe-4ecc-a9c1-d51b38b5f183">

## Дополнительные задания (со звёздочкой)

**Настоятельно рекомендуем выполнять все задания под звёздочкой.** Их выполнение поможет глубже разобраться в материале.   
Задания под звёздочкой необязательные к выполнению и не повлияют на получение зачёта по этому домашнему заданию. 

------
### Задание 2*. Установить HA кластер

1. Установить кластер в режиме HA.
2. Использовать нечётное количество Master-node.
3. Для cluster ip использовать keepalived или другой способ.

Преподаватели и аспиранты не захотели помогать. Оставляю тут инструкцию по установке

# High availability kubernetes API

На рисунке показано что и как будет взаимодействовать.

<img width="529" alt="HA" src="https://github.com/user-attachments/assets/bff71a65-dee5-4717-b932-4a79e2b38621">


В первую очередь нам понадобится кластерный IP адрес (в нашем примере это 192.168.219.189). 
Этот IP будет подниматься на одной из control node кластера. Если нода, на которой он был включён, по какой-то причине
будет выключена, этот IP будет поднят на другой control ноде. За этот функционал отвечает приложение 
[keepalived](https://keepalived.org), которое необходимо установить и настроить на каждой control ноде.

Все приложения (за пределами кластера kubernetes), которые захотят обращаться к kubernetes API, будут обращаться
на кластерный IP.

Второе приложение, которе нам понадобится - это [haproxy](https://www.haproxy.org/). Этот прокси сервер будет слушать
запросы на кластерном IP и пересылать их на реальные IP адреса машин, где физически находится kubernetes API.

Haproxy пересылает пакеты на машины по принципу round robbin, т.е. по очереди. Он контролирует доступность серверов.
И, если какой-либо сервер будет недоступен, он перестанет пересылать пакеты на него. 

Согласно нашей схемы, клиенты будут присылать запросы на 192.168.218.198:7443. Haproxy, запущенный на каждой
control ноде кластера, будет их пересылать на эти ноды, но на реальный IP адрес и на порт 6443.

Таким образом, нам придётся настроить и включить два приложения: keepalived и haproxy.

Потенциально их можно в дальнейшем запустить в кластере kubernetes или в виде контейнеров в containerd. Но я
думаю, что не стоит так заморачиваться. Эти приложения относятся к разряду "один раз настроил и забыл". Поэтому мы
поставим их как обычные приложения сервера. Но не забудем потом, при установке кластера, зарезервировать ресурсы под
такие системные приложения.

## keepalived

Установим приложение:

```shell
dnf install -y keepalived
```

На первой control ноде кластера, где будет располагаться мастер keepalived, отредактируем файл
`/etc/keepalived/keepalived.conf`:

```
vrrp_script chk_haproxy {
  script "killall -0 haproxy" # check the haproxy process
  interval 2                  # every 2 seconds
  weight 2                    # add 2 points if OK
}

vrrp_instance VI_1 {
  interface ens33            # interface to monitor
  state MASTER                # MASTER on master, BACKUP on slaves

  virtual_router_id 51
  priority 101                # 101 on master, 100 on slaves

  virtual_ipaddress {
    192.168.218.189/24          # virtual ip address
  }

  track_script {
    chk_haproxy
  }
}
```

На остальных conrol нодах это файл будет выглядеть так:

```
vrrp_script chk_haproxy {
  script "killall -0 haproxy" # check the haproxy process
  interval 2                  # every 2 seconds
  weight 2                    # add 2 points if OK
}

vrrp_instance VI_1 {
  interface ens33            # interface to monitor
  state BACKUP                # MASTER on master, BACKUP on slaves

  virtual_router_id 51
  priority 100                # 101 on master, 100 on slaves

  virtual_ipaddress {
    192.168.218.189/24          # virtual ip address
  }

  track_script {
    chk_haproxy
  }
}
```

Запустим keepalived:

```shell
systemctl enable keepalived
systemctl start keepalived
systemctl status keepalived
```

Проверим, появился ли в сети кластерный IP и, если появился, на какой машине он приземлился. 

```shell
ping -c4 192.168.218.189
ip n l
```

Если всё будет верно, то мы увидим MAC адрес сетевого интерфейса первой control кластера kubernetes.

## haproxy

Установим приложение:

```shell
dnf install -y haproxy
```

Отредактируем конфигурационный файл приложения `/etc/haproxy/haproxy.cfg`:

```
global
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

defaults
    log     global
    mode    tcp
    retries 2
    timeout client 30m
    timeout connect 4s
    timeout server 30m
    timeout check 5s

frontend main
    bind 192.168.218.189:7443
    default_backend             app

backend app
    balance     roundrobin
    server control1 192.168.218.171:6443 check
    server control2 192.168.218.172:6443 check
    server control3 192.168.218.173:6443 check

listen stats
    bind *:9000
    mode http
    stats enable  # Enable stats page
    stats hide-version  # Hide HAProxy version
    stats realm Haproxy\ Statistics  # Title text for popup window
    stats uri /haproxy_stats  # Stats URI
```

Включим haproxy:

```shell
systemctl enable haproxy
systemctl start haproxy
systemctl status haproxy
```

Поскольку у нас включен модуль статистики, мы можем к нему подключиться. Для этого в браузере откройте следующую
страницу `http://192.168.218.189:9000/haproxy_stats`.

## Немного автоматизации

Ansible [install-ha.yaml](https://github.com/BigKAA/00-kube-ansible/blob/main/services/install-ha.yaml)


### Правила приёма работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl get nodes`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
