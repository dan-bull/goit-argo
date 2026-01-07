Цей репозиторій демонструє GitOps-підхід із використанням **ArgoCD**, розгорнутого через **Terraform + Helm**, та автоматичний деплой застосунку з Git у Kubernetes.

> Для навчальних цілей використовується **локальний Kubernetes-кластер (kind)**, який імітує роботу з EKS.

---

## Архітектура рішення

- Kubernetes (kind)
- Terraform
- Helm
- ArgoCD
- GitHub (GitOps source of truth)

Потік:
```

Git push → ArgoCD → Kubernetes → Pods

```

---

## Структура репозиторію

```

goit-argo/
├── application.yaml
├── namespaces
│   ├── application
│   │   ├── ns.yaml
│   │   └── nginx.yaml
│   └── infra-tools
│       └── ns.yaml
└── README.md

````

---

## Запуск Terraform (ArgoCD)

### 1. Перейти в директорію Terraform

```bash
cd terraform/argocd
````

### 2. Ініціалізувати Terraform

```bash
terraform init
```

### 3. Застосувати конфігурацію

```bash
terraform apply
```

Підтвердити виконання:

```text
yes
```

---

## Перевірка, що ArgoCD працює

```bash
kubectl get pods -n infra-tools
```

Очікувані pod-и:

* argocd-server
* argocd-repo-server
* argocd-application-controller

---

## Доступ до ArgoCD UI

ArgoCD запущено з параметром `--insecure`, тому UI доступний **по HTTP**.

### Port-forward:

```bash
kubectl port-forward svc/argocd-server -n infra-tools 8080:80
```

### Відкрити в браузері:

```
http://localhost:8080
```

---

## Логін в ArgoCD

**Username:**

```
admin
```

**Password:**
(встановлюється вручну або через secret під час інсталяції)

---

## ArgoCD Application (GitOps)

Файл **`application.yaml`** описує ArgoCD Application і зберігається в корені цього репозиторію.

ArgoCD:

* підключається до цього Git-репозиторію
* автоматично синхронізує ресурси
* створює namespace
* застосовує manifests

### Застосування першого Application (bootstrap):

```bash
kubectl apply -f application.yaml -n infra-tools
```

---

## Перевірка синхронізації

### Перевірити Application:

```bash
kubectl get applications -n infra-tools
```

Очікуваний статус:

```
Synced | Healthy
```

### Перевірити pod-и застосунку:

```bash
kubectl get pods -n application
```

Очікувано:

```
nginx-xxxxx   Running
```

---

## Перевірка деплою

Застосунок `nginx` автоматично створюється ArgoCD після `git push`.

Будь-які зміни в:

```
namespaces/application/
```

→ автоматично застосовуються в кластері.

---

## Видалення ресурсів (cleanup)

Після завершення роботи:

```bash
terraform destroy
```

