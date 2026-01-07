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
<img width="698" height="400" alt="Get Nodes" src="https://github.com/user-attachments/assets/0b39af50-a833-4276-8d2a-116d1c454946" />

### 2. Ініціалізувати Terraform

```bash
terraform init
```

### 3. Застосувати конфігурацію

```bash
terraform apply
```
<img width="1499" height="1137" alt="terraform apply" src="https://github.com/user-attachments/assets/452505a1-c832-4f8a-a6bc-970178a47dc7" />

Підтвердити виконання:

```text
yes
```

---

## Перевірка, що ArgoCD працює

```bash
kubectl get pods -n infra-tools
```
<img width="644" height="161" alt="ARGOcd results" src="https://github.com/user-attachments/assets/4aea52ed-a643-4932-9e46-860a83e1c0b9" />

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
<img width="2056" height="1167" alt="browser" src="https://github.com/user-attachments/assets/7b6e0911-f268-4e26-b8d5-0c2616d66299" />

---

## Логін в ArgoCD

**Username:**

```
admin
```

**Password:**
(встановлюється вручну або через secret під час інсталяції)
<img width="2056" height="1165" alt="Screenshot 2026-01-07 at 18 02 38" src="https://github.com/user-attachments/assets/9a762d36-ed34-4932-89f9-8f111a85217c" />

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
<img width="747" height="586" alt="Screenshot 2026-01-07 at 18 14 10" src="https://github.com/user-attachments/assets/1cfa2e58-1a5c-48b5-83e7-f6760808a74c" />

---

## Перевірка синхронізації

### Перевірити Application:

```bash
kubectl get applications -n infra-tools
```
<img width="743" height="177" alt="Screenshot 2026-01-07 at 18 22 27" src="https://github.com/user-attachments/assets/5bf69d6e-3feb-4b91-864f-b6faacef09c6" />

Очікуваний статус:

```
Synced | Healthy
```
<img width="2053" height="1167" alt="Screenshot 2026-01-07 at 18 22 15" src="https://github.com/user-attachments/assets/7a8d5a52-06fb-430d-8b76-0a64000ffe29" />

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

