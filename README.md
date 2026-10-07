# Дипломный практикум в Yandex.Cloud, Марченко Николай
  * [Цели:](#цели)
  * [Этапы выполнения:](#этапы-выполнения)
     * [Создание облачной инфраструктуры](#создание-облачной-инфраструктуры)
     * [Создание Kubernetes кластера](#создание-kubernetes-кластера)
     * [Создание тестового приложения](#создание-тестового-приложения)
     * [Подготовка cистемы мониторинга и деплой приложения](#подготовка-cистемы-мониторинга-и-деплой-приложения)
     * [Установка и настройка CI/CD](#установка-и-настройка-cicd)
  * [Что необходимо для сдачи задания?](#что-необходимо-для-сдачи-задания)
  * [Как правильно задавать вопросы дипломному руководителю?](#как-правильно-задавать-вопросы-дипломному-руководителю)

**Перед началом работы над дипломным заданием изучите [Инструкция по экономии облачных ресурсов](https://github.com/netology-code/devops-materials/blob/master/cloudwork.MD).**

---
## Цели:

1. Подготовить облачную инфраструктуру на базе облачного провайдера Яндекс.Облако.
2. Запустить и сконфигурировать Kubernetes кластер.
3. Установить и настроить систему мониторинга.
4. Настроить и автоматизировать сборку тестового приложения с использованием Docker-контейнеров.
5. Настроить CI для автоматической сборки и тестирования.
6. Настроить CD для автоматического развёртывания приложения.

---
## Этапы выполнения:


### Создание облачной инфраструктуры

Для начала необходимо подготовить облачную инфраструктуру в ЯО при помощи [Terraform](https://www.terraform.io/).

Особенности выполнения:

- Бюджет купона ограничен, что следует иметь в виду при проектировании инфраструктуры и использовании ресурсов;
Для облачного k8s используйте региональный мастер(неотказоустойчивый). Для self-hosted k8s минимизируйте ресурсы ВМ и долю ЦПУ. В обоих вариантах используйте прерываемые ВМ для worker nodes.

Предварительная подготовка к установке и запуску Kubernetes кластера.

1. Создайте сервисный аккаунт, который будет в дальнейшем использоваться Terraform для работы с инфраструктурой с необходимыми и достаточными правами. Не стоит использовать права суперпользователя
2. Подготовьте [backend](https://developer.hashicorp.com/terraform/language/backend) для Terraform:  
   а. Рекомендуемый вариант: S3 bucket в созданном ЯО аккаунте(создание бакета через TF)
   б. Альтернативный вариант:  [Terraform Cloud](https://app.terraform.io/)
3. Создайте конфигурацию Terrafrom, используя созданный бакет ранее как бекенд для хранения стейт файла. Конфигурации Terraform для создания сервисного аккаунта и бакета и основной инфраструктуры следует сохранить в разных папках.
4. Создайте VPC с подсетями в разных зонах доступности.
5. Убедитесь, что теперь вы можете выполнить команды `terraform destroy` и `terraform apply` без дополнительных ручных действий.
6. В случае использования [Terraform Cloud](https://app.terraform.io/) в качестве [backend](https://developer.hashicorp.com/terraform/language/backend) убедитесь, что применение изменений успешно проходит, используя web-интерфейс Terraform cloud.

Ожидаемые результаты:

1. Terraform сконфигурирован и создание инфраструктуры посредством Terraform возможно без дополнительных ручных действий, стейт основной конфигурации сохраняется в бакете или Terraform Cloud
2. Полученная конфигурация инфраструктуры является предварительной, поэтому в ходе дальнейшего выполнения задания возможны изменения.

---
### Создание Kubernetes кластера

На этом этапе необходимо создать [Kubernetes](https://kubernetes.io/ru/docs/concepts/overview/what-is-kubernetes/) кластер на базе предварительно созданной инфраструктуры.   Требуется обеспечить доступ к ресурсам из Интернета.

Это можно сделать двумя способами:

1. Рекомендуемый вариант: самостоятельная установка Kubernetes кластера.  
   а. При помощи Terraform подготовить как минимум 3 виртуальных машины Compute Cloud для создания Kubernetes-кластера. Тип виртуальной машины следует выбрать самостоятельно с учётом требовании к производительности и стоимости. Если в дальнейшем поймете, что необходимо сменить тип инстанса, используйте Terraform для внесения изменений.  
   б. Подготовить [ansible](https://www.ansible.com/) конфигурации, можно воспользоваться, например [Kubespray](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/)  
   в. Задеплоить Kubernetes на подготовленные ранее инстансы, в случае нехватки каких-либо ресурсов вы всегда можете создать их при помощи Terraform.
2. Альтернативный вариант: воспользуйтесь сервисом [Yandex Managed Service for Kubernetes](https://cloud.yandex.ru/services/managed-kubernetes)  
  а. С помощью terraform resource для [kubernetes](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/kubernetes_cluster) создать **региональный** мастер kubernetes с размещением нод в разных 3 подсетях      
  б. С помощью terraform resource для [kubernetes node group](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/kubernetes_node_group)
  
Ожидаемый результат:

1. Работоспособный Kubernetes кластер.
2. В файле `~/.kube/config` находятся данные для доступа к кластеру.
3. Команда `kubectl get pods --all-namespaces` отрабатывает без ошибок.

---
### Создание тестового приложения

Для перехода к следующему этапу необходимо подготовить тестовое приложение, эмулирующее основное приложение разрабатываемое вашей компанией.

Способ подготовки:

1. Рекомендуемый вариант:  
   а. Создайте отдельный git репозиторий с простым nginx конфигом, который будет отдавать статические данные.  
   б. Подготовьте Dockerfile для создания образа приложения.  
2. Альтернативный вариант:  
   а. Используйте любой другой код, главное, чтобы был самостоятельно создан Dockerfile.

Ожидаемый результат:

1. Git репозиторий с тестовым приложением и Dockerfile.
2. Регистри с собранным docker image. В качестве регистри может быть DockerHub или [Yandex Container Registry](https://cloud.yandex.ru/services/container-registry), созданный также с помощью terraform.

---
### Подготовка cистемы мониторинга и деплой приложения

Уже должны быть готовы конфигурации для автоматического создания облачной инфраструктуры и поднятия Kubernetes кластера.  
Теперь необходимо подготовить конфигурационные файлы для настройки нашего Kubernetes кластера.

Цель:
1. Задеплоить в кластер [prometheus](https://prometheus.io/), [grafana](https://grafana.com/), [alertmanager](https://github.com/prometheus/alertmanager), [экспортер](https://github.com/prometheus/node_exporter) основных метрик Kubernetes.
2. Задеплоить тестовое приложение, например, [nginx](https://www.nginx.com/) сервер отдающий статическую страницу.

Способ выполнения:
1. Воспользоваться пакетом [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus), который уже включает в себя [Kubernetes оператор](https://operatorhub.io/) для [grafana](https://grafana.com/), [prometheus](https://prometheus.io/), [alertmanager](https://github.com/prometheus/alertmanager) и [node_exporter](https://github.com/prometheus/node_exporter). Альтернативный вариант - использовать набор helm чартов от [bitnami](https://github.com/bitnami/charts/tree/main/bitnami).

### Деплой инфраструктуры в terraform pipeline

1. Если на первом этапе вы не воспользовались [Terraform Cloud](https://app.terraform.io/), то задеплойте и настройте в кластере [atlantis](https://www.runatlantis.io/) для отслеживания изменений инфраструктуры. Альтернативный вариант 3 задания: вместо Terraform Cloud или atlantis настройте на автоматический запуск и применение конфигурации terraform из вашего git-репозитория в выбранной вами CI-CD системе при любом комите в main ветку. Предоставьте скриншоты работы пайплайна из CI/CD системы.

Ожидаемый результат:
1. Git репозиторий с конфигурационными файлами для настройки Kubernetes.
2. Http доступ на 80 порту к web интерфейсу grafana.
3. Дашборды в grafana отображающие состояние Kubernetes кластера.
4. Http доступ на 80 порту к тестовому приложению.
5. Atlantis или terraform cloud или ci/cd-terraform
---
### Установка и настройка CI/CD

Осталось настроить ci/cd систему для автоматической сборки docker image и деплоя приложения при изменении кода.

Цель:

1. Автоматическая сборка docker образа при коммите в репозиторий с тестовым приложением.
2. Автоматический деплой нового docker образа.

Можно использовать [teamcity](https://www.jetbrains.com/ru-ru/teamcity/), [jenkins](https://www.jenkins.io/), [GitLab CI](https://about.gitlab.com/stages-devops-lifecycle/continuous-integration/) или GitHub Actions.

Ожидаемый результат:

1. Интерфейс ci/cd сервиса доступен по http.
2. При любом коммите в репозиторие с тестовым приложением происходит сборка и отправка в регистр Docker образа.
3. При создании тега (например, v1.0.0) происходит сборка и отправка с соответствующим label в регистри, а также деплой соответствующего Docker образа в кластер Kubernetes.

---
## Что необходимо для сдачи задания?

1. Репозиторий с конфигурационными файлами Terraform и готовность продемонстрировать создание всех ресурсов с нуля.
2. Пример pull request с комментариями созданными atlantis'ом или снимки экрана из Terraform Cloud или вашего CI-CD-terraform pipeline.
3. Репозиторий с конфигурацией ansible, если был выбран способ создания Kubernetes кластера при помощи ansible.
4. Репозиторий с Dockerfile тестового приложения и ссылка на собранный docker image.
5. Репозиторий с конфигурацией Kubernetes кластера.
6. Ссылка на тестовое приложение и веб интерфейс Grafana с данными доступа.
7. Все репозитории рекомендуется хранить на одном ресурсе (github, gitlab)

__________________________________________________________________

# Решение:

## О проекте

Полный цикл DevOps: от создания облачной инфраструктуры через Terraform до автоматического CI/CD с деплоем в Kubernetes.

Стек технологий:
- **Terraform** — инфраструктура как код
- **Kubespray** — установка self-hosted Kubernetes
- **NGINX Ingress Controller** — маршрутизация трафика
- **kube-prometheus** — мониторинг (Prometheus, Grafana, Alertmanager)
- **GitLab CI/CD** — автоматизация сборки и деплоя
- **Yandex Container Registry** — хранение Docker-образов

---

## 📁 Структура репозиториев

Все репозитории на **GitLab** (или GitHub):

| Репозиторий | Описание | Ссылка |
|-------------|----------|--------|
| **terraform** | Инфраструктура + Terraform pipeline | `gitlab.com/Doskaks/terraform` |
| **test-app** | Тестовое приложение + CI/CD | `gitlab.com/Doskaks/test-app` |

**Локальная структура проекта:**

```
devops-diplom-yandexcloud/
├── kube-prometheus/          # Чужой репозиторий (клонируется отдельно)
├── kubespray/                # Чужой репозиторий (клонируется отдельно)
├── terraform/                # ← Репозиторий terraform
│   ├── bootstrap/            # S3, KMS, Container Registry
│   │   ├── .gitignore
│   │   ├── main.tf
│   │   ├── outputs.tf
│   │   ├── variables.tf
│   │   └── versions.tf
│   ├── infrastructure/       # VPC, ВМ, K8s
│   │   ├── .gitignore
│   │   ├── backend.tf
│   │   ├── compute.tf
│   │   ├── inventory.tf
│   │   ├── kubespray_vars.tf
│   │   ├── nat.tf
│   │   ├── outputs.tf
│   │   ├── providers.tf
│   │   ├── security-groups.tf
│   │   ├── service-accounts.tf
│   │   ├── static-ip.tf
│   │   ├── variables.tf
│   │   ├── versions.tf
│   │   ├── vpc.tf
│   │   ├── id_rsa.pub        # Публичный SSH-ключ (для CI)
│   │   └── templates/
│   │       ├── hosts.yaml.tpl
│   │       └── k8s-cluster-extra.yml.tpl
│   ├── .gitignore
│   ├── .gitlab-ci.yml        # Terraform pipeline
│   └── .terraformrc          # Зеркало Terraform
│
└── test-app/                 # ← Репозиторий test-app
    ├── k8s/
    │   ├── deployment.yaml
    │   ├── ingress.yaml
    │   ├── service.yaml
    │   └── monitoring/
    │       └── grafana-ingress.yaml
    ├── .gitignore
    ├── .gitlab-ci.yml        # CI/CD pipeline
    ├── Dockerfile
    ├── index.html
    ├── nginx.conf
    ├── README.md
    ├── script.js
    └── style.css
```

---

## 🎯 Цели проекта

1. ✅ Подготовить облачную инфраструктуру на базе **Яндекс.Облако**
2. ✅ Запустить и сконфигурировать **Kubernetes кластер**
3. ✅ Установить и настроить **систему мониторинга**
4. ✅ Настроить и автоматизировать сборку тестового приложения с использованием **Docker-контейнеров**
5. ✅ Настроить **CI** для автоматической сборки и тестирования
6. ✅ Настроить **CD** для автоматического развёртывания приложения

---

## 📋 Этап 1. Создание облачной инфраструктуры

### 1.1. Ручной bootstrap (консоль Yandex Cloud)

**Создание сервисного аккаунта:**

1. Yandex Cloud → **Identity and Access Management** → **Сервисные аккаунты** → **Создать**.
2. Имя: `terraform-sa`.
3. Назначить роли:
   - `k8s.editor`
   - `iam.serviceAccounts.user`
   - `iam.serviceAccounts.admin`
   - `vpc.privateAdmin`
   - `vpc.publicAdmin`
   - `vpc.securityGroups.admin`
   - `storage.admin`
   - `container-registry.admin`
   - `kms.editor`

**Создание ключей:**

- **Авторизованный ключ** → `~/.yc/authorized_key.json` (chmod 600).
- **Статический ключ** → `~/.yc/secrets/static-key.txt` (chmod 600).

**Настройка CLI:**

```bash
yc config profile create sa-profile
yc config profile activate sa-profile
yc config set service-account-key ~/.yc/authorized_key.json
yc config set cloud-id <cloud_id>
yc config set folder-id <folder_id>
```

**Настройка зеркала Terraform** (`~/.terraformrc`):

```hcl
provider_installation {
  network_mirror {
    url = "https://terraform-mirror.yandexcloud.net/"
    include = [
      "registry.terraform.io/yandex-cloud/yandex",
      "registry.terraform.io/hashicorp/null",
      "registry.terraform.io/hashicorp/local"
    ]
  }
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}
```

### 1.2. Terraform bootstrap

**Цель:** создать S3-бакет для стейта, KMS-ключ, Container Registry.

```bash
cd terraform/bootstrap
terraform init
terraform apply
```

**Создаётся:**

| Ресурс | Назначение |
|--------|-----------|
| `yandex_kms_symmetric_key` | Шифрование бакета |
| `yandex_storage_bucket` | Хранение `terraform.tfstate` |
| `yandex_container_registry` | Docker-образы приложения |

**Сохранённые значения:**
- `state_bucket_name` — имя бакета для backend.
- `container_registry_id` — ID реестра.

### 1.3. Terraform infrastructure

**Цель:** создать VPC, 3 ВМ, статический IP, security group.

```bash
cd terraform/infrastructure
terraform init
terraform apply
```

**Создаётся:**

| Ресурс | Назначение |
|--------|-----------|
| `yandex_vpc_network` | VPC-сеть |
| `yandex_vpc_subnet` × 3 | Подсети в зонах a, b, d |
| `yandex_vpc_security_group` | Правила для K8s, SSH, HTTP, NodePort |
| `yandex_vpc_gateway` + `route_table` | NAT для воркеров |
| `yandex_vpc_address` | **Статический IP** для мастера |
| `yandex_compute_instance.k8s_master` | Мастер-нода (standard-v3, preemptible) |
| `yandex_compute_instance.k8s_workers` × 2 | Worker-ноды |
| `yandex_iam_service_account` × 2 | SA для мастера и нод |
| `null_resource.ansible_inventory` | Генерация `hosts.yaml` |
| `null_resource.kubespray_k8s_cluster_extra` | Генерация `k8s-cluster-extra.yml` |

**Backend** — S3-бакет из bootstrap:

```hcl
terraform {
  backend "s3" {
    endpoint = "storage.yandexcloud.net"
    bucket   = "terraform-state-<folder_id>"
    key      = "infrastructure/terraform.tfstate"
    region   = "ru-central1"
    skip_region_validation      = true
    skip_credentials_validation = true
    skip_requesting_account_id  = true
    skip_s3_checksum            = true
  }
}
```

**Ожидаемый результат:**
- ✅ Terraform создаёт инфраструктуру без ручных действий.
- ✅ Стейт хранится в S3-бакете.
- ✅ `terraform destroy` + `terraform apply` работают без ручных правок.

---

## 📋 Этап 2. Создание Kubernetes кластера

### 2.1. Установка Kubespray

```bash
cd ~/Дипломный\ проект/devops-diplom-yandexcloud
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray

# Создать venv с Python 3.11
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 2.2. Подготовка inventory

```bash
# Скопировать sample
cp -rfp inventory/sample inventory/mycluster

# Скопировать group_vars
cp -rfp inventory/sample/group_vars inventory/mycluster/

# Проверить hosts.yaml (генерируется Terraform)
cat inventory/mycluster/hosts.yaml
```

**`hosts.yaml`** — генерируется Terraform:
- `k8s-master-1` → публичный IP (bastion).
- `k8s-worker-1/2` → внутренние IP.
- ProxyJump через мастер.

### 2.3. Запуск Kubespray

```bash
source .venv/bin/activate

# Проверка
ansible -i inventory/mycluster/hosts.yaml all -m ping

# Установка K8s (15-25 мин)
ansible-playbook -i inventory/mycluster/hosts.yaml --become --become-user=root cluster.yml
```

### 2.4. Исправление SAN сертификата (баг Kubespray v2.28)

**Проблема:** в SAN сертификата нет публичного IP.

**Решение:** плейбук `fix-cert.yml`:

```yaml
---
- name: Перегенерация сертификата API-сервера
  hosts: kube_control_plane
  become: yes
  vars:
    master_public_ip: "{{ hostvars[inventory_hostname]['ansible_host'] }}"
    master_internal_ip: "{{ hostvars[inventory_hostname]['ip'] }}"
    k8s_version: "v1.32.8"

  tasks:
    - name: Создать конфиг kubeadm
      copy:
        dest: /tmp/kubeadm-certs-fix.yaml
        content: |
          apiVersion: kubeadm.k8s.io/v1beta4
          kind: ClusterConfiguration
          kubernetesVersion: {{ k8s_version }}
          apiServer:
            certSANs:
            - "{{ master_public_ip }}"
            - "{{ master_internal_ip }}"
            - "10.233.0.1"
            - "127.0.0.1"
            - "::1"
            - "k8s-master-1"
            - "kubernetes"
            - "kubernetes.default"
            - "kubernetes.default.svc"
            - "kubernetes.default.svc.cluster.local"
            - "localhost"

    - name: Удалить старые сертификаты
      file:
        path: "{{ item }}"
        state: absent
      loop:
        - /etc/kubernetes/pki/apiserver.crt
        - /etc/kubernetes/pki/apiserver.key

    - name: Сгенерировать новый сертификат
      command: kubeadm init phase certs apiserver --config /tmp/kubeadm-certs-fix.yaml

    - name: Найти ID пода
      shell: crictl pods | grep kube-apiserver | awk '{print $1}' | head -1
      register: api_pod_id

    - name: Перезапустить API-сервер
      shell: |
        crictl stopp {{ api_pod_id.stdout }}
        crictl rmp {{ api_pod_id.stdout }}
      when: api_pod_id.stdout != ""

    - name: Обновить ConfigMap
      command: kubeadm init phase upload-config kubeadm --config /tmp/kubeadm-certs-fix.yaml
```

**Запуск:**

```bash
ansible-playbook -i inventory/mycluster/hosts.yaml fix-cert.yml
```

### 2.5. Получение kubeconfig

```bash
MASTER_IP=$(cd ../terraform/infrastructure && terraform output -raw master_static_ip)

mkdir -p ~/.kube
ssh -i ~/.ssh/id_rsa ubuntu@$MASTER_IP "sudo cat /etc/kubernetes/admin.conf" > ~/.kube/config
chmod 600 ~/.kube/config
sed -i "s|server: https://[0-9.]*:6443|server: https://${MASTER_IP}:6443|" ~/.kube/config
```

### 2.6. Проверка

```bash
kubectl get nodes
# NAME           STATUS   ROLES           AGE   VERSION
# k8s-master-1   Ready    control-plane   19m   v1.32.8
# k8s-worker-1   Ready    <none>          18m   v1.32.8
# k8s-worker-2   Ready    <none>          18m   v1.32.8

kubectl get pods --all-namespaces
```

**Ожидаемый результат:**
- ✅ Работоспособный K8s кластер (3 ноды).
- ✅ `~/.kube/config` настроен.
- ✅ `kubectl get pods --all-namespaces` работает.

---

## 📋 Этап 3. Создание тестового приложения

### 3.1. Приложение

**Файлы приложения** (`test-app/`):

| Файл | Назначение |
|------|-----------|
| `index.html` | Калькулятор асфальтирования (статическая страница) |
| `style.css` | Стили |
| `script.js` | Логика калькулятора |
| `nginx.conf` | Конфиг nginx (порт 80, `/health`) |
| `Dockerfile` | Сборка образа на базе `nginx:alpine` |
| `README.md` | Описание |

**`Dockerfile`:**

```dockerfile
FROM nginx:alpine

LABEL maintainer="Nikolay"
LABEL description="Калькулятор асфальтирования — дипломный проект DevOps"

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY index.html /usr/share/nginx/html/index.html
COPY style.css /usr/share/nginx/html/style.css
COPY script.js /usr/share/nginx/html/script.js

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost/health || exit 1

CMD ["nginx", "-g", "daemon off;"]
```

### 3.2. Сборка и push образа

```bash
cd ~/Дипломный\ проект/devops-diplom-yandexcloud/test-app

# Аутентификация
yc container registry configure-docker

# Сборка
docker build -t cr.yandex/crpsj2ejeasjt6e1fna1/test-app:v1.0.0 .

# Push
docker push cr.yandex/crpsj2ejeasjt6e1fna1/test-app:v1.0.0
```

### 3.3. Git-репозиторий

```bash
git init
git branch -M main
git remote add origin https://gitlab.com/Doskaks/test-app.git

git add .
git commit -m "Initial commit: test-app calculator"
git push -u origin main
```

**Ожидаемый результат:**
- ✅ Git-репозиторий `test-app`.
- ✅ Docker-образ в **Yandex Container Registry**.

---

## 📋 Этап 4. Мониторинг и деплой приложения

### 4.1. Установка NGINX Ingress Controller

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.kind=DaemonSet \
  --set controller.hostPort.enabled=true \
  --set controller.hostPort.ports.http=80 \
  --set controller.hostPort.ports.https=443 \
  --set controller.tolerations[0].key=node-role.kubernetes.io/control-plane \
  --set controller.tolerations[0].operator=Exists \
  --set controller.tolerations[0].effect=NoSchedule
```

**Проверка:**

```bash
kubectl get pods -n ingress-nginx -o wide
# 3 пода Running на всех нодах (включая мастер)
```

### 4.2. Деплой test-app

```bash
cd ~/Дипломный\ проект/devops-diplom-yandexcloud/test-app

kubectl create namespace test-app

kubectl create secret docker-registry yandex-registry-secret \
  --namespace test-app \
  --docker-server=cr.yandex \
  --docker-username=json_key \
  --docker-password="$(cat ~/.yc/authorized_key.json)" \
  --docker-email=unused

kubectl apply -f k8s/
```

**Проверка:**

```bash
kubectl get pods -n test-app
kubectl get ingress -n test-app
curl -I http://62.84.118.27/
# HTTP/1.1 200 OK
```

### 4.3. Установка kube-prometheus

```bash
cd ~/Дипломный\ проект/devops-diplom-yandexcloud
git clone https://github.com/prometheus-operator/kube-prometheus.git
cd kube-prometheus

kubectl apply --server-side -f manifests/setup
kubectl wait --for condition=Established --all CustomResourceDefinition --namespace=monitoring
kubectl apply -f manifests/
```

**Проверка:**

```bash
kubectl get pods -n monitoring
# Все поды Running: Prometheus, Grafana, Alertmanager, Node Exporter
```

### 4.4. Настройка Grafana

**Удалить NetworkPolicy:**

```bash
kubectl delete networkpolicy grafana -n monitoring
```

**Настроить `grafana.ini`:**

```bash
cat > /tmp/grafana.ini <<'EOF'
[server]
root_url = https://62.84.118.27/grafana
serve_from_sub_path = true
EOF

GRAFANA_INI_B64=$(cat /tmp/grafana.ini | base64 -w0)
kubectl patch secret grafana-config -n monitoring \
  -p "{\"data\":{\"grafana.ini\":\"${GRAFANA_INI_B64}\"}}"

kubectl rollout restart deployment grafana -n monitoring
```

**Создать TLS-сертификат:**

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /tmp/tls.key -out /tmp/tls.crt \
  -subj "/CN=62.84.118.27" \
  -addext "subjectAltName=IP:62.84.118.27"

kubectl create secret tls grafana-tls \
  --cert=/tmp/tls.crt --key=/tmp/tls.key -n monitoring
```

**Создать Ingress** (`k8s/monitoring/grafana-ingress.yaml`):

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana
  namespace: monitoring
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - "62.84.118.27"
      secretName: grafana-tls
  rules:
    - http:
        paths:
          - path: /grafana
            pathType: Prefix
            backend:
              service:
                name: grafana
                port:
                  number: 3000
```

```bash
kubectl apply -f k8s/monitoring/grafana-ingress.yaml
```

### 4.5. Проверка

```bash
# Grafana
curl -k -I https://62.84.118.27/grafana/
# HTTP/2 302

# test-app
curl -I http://62.84.118.27/
# HTTP/1.1 200

# Prometheus (port-forward)
kubectl port-forward -n monitoring svc/prometheus-k8s 9090:9090
# http://localhost:9090
```

**Пароль Grafana:**

```bash
kubectl get secret grafana-config -n monitoring -o jsonpath='{.data.grafana\.ini}' | base64 -d
```

**Ожидаемый результат:**
- ✅ HTTP-доступ к Grafana на 80 порту.
- ✅ Дашборды K8s.
- ✅ HTTP-доступ к test-app на 80 порту.
- ✅ CI/CD-terraform pipeline.

---

## 📋 Этап 5. Terraform pipeline

### 5.1. GitLab Variables

GitLab → `terraform` → **Настройки** → **CI/CD** → **Переменные**:

| Key | Value | Type | Visibility | Protect |
|-----|-------|------|-----------|---------|
| `YC_KEY` | base64 от `authorized_key.json` | Variable | Visible | ❌ |
| `AWS_ACCESS_KEY_ID` | из `static-key.txt` | Variable | Visible | ❌ |
| `AWS_SECRET_ACCESS_KEY` | из `static-key.txt` | Variable | Visible | ❌ |
| `TF_VAR_cloud_id` | `yc config get cloud-id` | Variable | Visible | ❌ |
| `TF_VAR_folder_id` | `yc config get folder-id` | Variable | Visible | ❌ |
| `TF_VAR_ssh_public_key_path` | `${CI_PROJECT_DIR}/infrastructure/id_rsa.pub` | Variable | Visible | ❌ |
| `TF_VAR_service_account_key_file` | `/root/.yc/authorized_key.json` | Variable | Visible | ❌ |

**Получить base64:**

```bash
cat ~/.yc/authorized_key.json | jq -c . | base64 -w0
```

### 5.2. `.gitlab-ci.yml`

```yaml
---
stages:
  - validate
  - plan
  - apply

variables:
  TF_ROOT: "${CI_PROJECT_DIR}/infrastructure"
  TF_IN_AUTOMATION: "true"
  TF_INPUT: "false"
  TF_CLI_CONFIG_FILE: "${CI_PROJECT_DIR}/.terraformrc"

default:
  image:
    name: hashicorp/terraform:1.9.7
    entrypoint: [""]
  before_script:
    - mkdir -p ~/.yc
    - echo "$YC_KEY" | base64 -d > ~/.yc/authorized_key.json
    - chmod 600 ~/.yc/authorized_key.json
    - export YC_SERVICE_ACCOUNT_KEY_FILE=~/.yc/authorized_key.json
    - cd "${TF_ROOT}"
    - terraform init

validate:
  stage: validate
  script:
    - terraform fmt --recursive --check
    - terraform validate
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

plan:
  stage: plan
  script:
    - terraform plan -out=tfplan
  artifacts:
    paths:
      - ${TF_ROOT}/tfplan
    expire_in: 1 day
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

apply:
  stage: apply
  dependencies:
    - plan
  script:
    - terraform apply -auto-approve tfplan
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: manual
      allow_failure: false
```

### 5.3. Self-hosted GitLab Runner

```bash
docker run -d --name gitlab-runner --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v gitlab-runner-config:/etc/gitlab-runner \
  gitlab/gitlab-runner:latest

# Регистрация
docker exec -it gitlab-runner gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --token "glrt-..." \
  --executor "docker" \
  --docker-image "alpine:latest" \
  --description "terraform-runner"
```

### 5.4. Push и проверка

```bash
cd terraform
git add .
git commit -m "Add Terraform CI/CD pipeline"
git push
```

**Ожидаемый результат:**
- ✅ `validate` — Passed.
- ✅ `plan` — Passed.
- ✅ `apply` — Manual.

---

## 📋 Этап 6. CI/CD для приложения

### 6.1. GitLab Variables

GitLab → `test-app` → **Настройки** → **CI/CD** → **Переменные**:

| Key | Value | Type | Visibility | Protect |
|-----|-------|------|-----------|---------|
| `YC_KEY` | base64 от `authorized_key.json` | Variable | Visible | ❌ |
| `REGISTRY_ID` | `crpsj2ejeasjt6e1fna1` | Variable | Visible | ❌ |
| `KUBE_CONFIG` | base64 от `~/.kube/config` | Variable | Visible | ❌ |

### 6.2. `.gitlab-ci.yml`

```yaml
---
stages:
  - build
  - deploy

build:
  stage: build
  image:
    name: docker:24
    entrypoint: [""]
  services:
    - docker:24-dind
  variables:
    DOCKER_HOST: "tcp://docker:2375"
    DOCKER_TLS_CERTDIR: ""
  before_script:
    - echo "$YC_KEY" | base64 -d > /tmp/key.json
    - cat /tmp/key.json | docker login --username json_key --password-stdin cr.yandex
    - export IMAGE_NAME="cr.yandex/${REGISTRY_ID}/test-app"
    - export IMAGE_TAG="${CI_COMMIT_TAG:-$CI_COMMIT_SHORT_SHA}"
  script:
    - docker build -t "${IMAGE_NAME}:${IMAGE_TAG}" .
    - docker push "${IMAGE_NAME}:${IMAGE_TAG}"
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
    - if: $CI_COMMIT_TAG

deploy:
  stage: deploy
  image:
    name: alpine/k8s:1.31.0
    entrypoint: [""]
  before_script:
    - mkdir -p ~/.kube
    - echo "$KUBE_CONFIG" | base64 -d > ~/.kube/config
    - chmod 600 ~/.kube/config
    - export IMAGE_NAME="cr.yandex/${REGISTRY_ID}/test-app"
    - export IMAGE_TAG="${CI_COMMIT_TAG:-$CI_COMMIT_SHORT_SHA}"
  script:
    - kubectl set image deployment/test-app test-app="${IMAGE_NAME}:${IMAGE_TAG}" -n test-app
    - kubectl rollout status deployment/test-app -n test-app --timeout=120s
  rules:
    - if: $CI_COMMIT_TAG
  environment:
    name: production
    url: http://62.84.118.27
```

### 6.3. Self-hosted Runner с `privileged`

```bash
docker exec -it gitlab-runner gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --token "glrt-..." \
  --executor "docker" \
  --docker-image "alpine:latest" \
  --description "test-app-runner"

# Включить privileged
docker exec -it gitlab-runner sh
sed -i 's/privileged = false/privileged = true/' /etc/gitlab-runner/config.toml
exit

docker restart gitlab-runner
```

### 6.4. Push и проверка

```bash
cd test-app

git add .gitlab-ci.yml
git commit -m "Add CI/CD pipeline"
git push

# Создать тег
git tag v1.0.0
git push origin v1.0.0
```

**Ожидаемый результат:**
- ✅ При коммите — `build` (сборка + push образа).
- ✅ При теге — `build` + `deploy` (деплой в K8s).

---

## 📌 Итоговые ссылки

| Сервис | Ссылка / Доступ |
|--------|-----------------|
| **test-app** | http://62.84.118.27/ |
| **Grafana** | https://62.84.118.27/grafana/ |
| **Логин/пароль Grafana** | `admin` / `prom-operator` |
| **Репозиторий `terraform`** | `gitlab.com/Doskaks/terraform` |
| **Репозиторий `test-app`** | `gitlab.com/Doskaks/test-app` |
| **Container Registry** | `cr.yandex/crpsj2ejeasjt6e1fna1/test-app` |

---

## ✅ Соответствие требованиям сдачи

| # | Требование | Статус |
|---|-----------|--------|
| 1 | Репозиторий с Terraform | ✅ `terraform` |
| 2 | Скриншоты CI/CD-terraform pipeline | ✅ Этап 5 |
| 3 | Репозиторий с Ansible | ⚠️ Не нужен (self-hosted Kubespray) |
| 4 | Репозиторий с Dockerfile + ссылка на образ | ✅ `test-app` |
| 5 | Репозиторий с конфигурацией K8s | ✅ `test-app/k8s/` |
| 6 | Ссылка на приложение и Grafana | ✅ см. выше |
| 7 | Все репозитории на одном ресурсе | ✅ GitLab |

---

## 🎯 Заключение

Проект **полностью автоматизирован**:

1. **Инфраструктура** — Terraform создаёт VPC, 3 ВМ, статический IP, KMS, S3, Container Registry.
2. **Kubernetes** — устанавливается через **Kubespray** (self-hosted, 3 ноды). Ingress Controller (NGINX, DaemonSet с tolerations).
3. **Приложение** — статический сайт (nginx + HTML/CSS/JS) в Docker-образе.
4. **Мониторинг** — **kube-prometheus** (Prometheus, Grafana, Alertmanager, Node Exporter).
5. **Terraform pipeline** — GitLab CI/CD `validate → plan → apply`.
6. **CI/CD приложения** — GitLab CI/CD `build → deploy` (при теге).