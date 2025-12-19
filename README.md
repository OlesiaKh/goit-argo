Цей проєкт демонструє розгортання ArgoCD у Kubernetes кластері (AWS EKS) за допомогою Terraform та використання GitOps-підходу для автоматичного деплою застосунків з Git-репозиторію.

У рамках завдання:
- ArgoCD розгорнуто в EKS як Helm release через Terraform
- Створено окремий Git-репозиторій з Kubernetes-маніфестами
- Налаштовано ArgoCD Applications для nginx та MLflow
- Після git push ArgoCD автоматично синхронізує стан кластера з Git

--------------------------------------------------
1. Розгортання ArgoCD через Terraform
--------------------------------------------------

Terraform-код для розгортання ArgoCD знаходиться в окремому проєкті:
terraform/argocd

ArgoCD встановлюється як helm_release у namespace infra-tools.
Усі значення Helm-чарту винесені у файл values/argocd-values.yaml.

Запуск Terraform:

terraform init
terraform apply

Перевірка, що ArgoCD pod-и запущені:

kubectl get pods -n infra-tools

У namespace infra-tools мають бути pod-и з префіксом argocd-.

--------------------------------------------------
2. Git-репозиторій з маніфестами
--------------------------------------------------

Цей репозиторій містить Kubernetes-маніфести та ArgoCD Applications.

Структура репозиторію:

goit-argo
├── namespaces
│   ├── application
│   │   ├── ns.yaml
│   │   └── nginx.yaml
│   └── infra-tools
│       └── ns.yaml
├── application.yaml
├── mlflow-application.yaml
├── README.md
└── argocdapplications.png

--------------------------------------------------
3. ArgoCD Application: nginx
--------------------------------------------------

Файл application.yaml описує ArgoCD Application для nginx.

- Джерело: GitHub репозиторій
- Path: namespaces/application
- Namespace: application
- Увімкнено auto-sync, prune та self-heal
- Namespace створюється автоматично (CreateNamespace=true)

Застосування Application у кластер:

kubectl apply -f application.yaml

Перевірка:

kubectl get applications -n infra-tools
kubectl get pods -n application

--------------------------------------------------
4. ArgoCD Application: MLflow (Helm)
--------------------------------------------------

MLflow розгортається як Helm-чарт через ArgoCD Application.
Опис знаходиться у файлі mlflow-application.yaml.

Основні параметри:
- repoURL: https://community-charts.github.io/helm-charts
- chart: mlflow
- targetRevision: 0.7.19
- namespace: application
- auto-sync та self-heal увімкнені
- Namespace створюється автоматично

Застосування MLflow Application:

kubectl apply -f mlflow-application.yaml

Перевірка:

kubectl get applications -n infra-tools
kubectl get pods -n application
kubectl get svc -n application

Очікувані сервіси:
- mlflow (порт 5000)
- nginx (порт 80)

--------------------------------------------------
5. Доступ до ArgoCD UI
--------------------------------------------------

Port-forward для ArgoCD:

kubectl port-forward svc/argocd-server -n infra-tools 8080:443

Відкрити у браузері:
https://localhost:8080

Отримати пароль admin:

kubectl -n infra-tools get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 --decode

Логін:
admin


--------------------------------------------------
7. Доступ до MLflow
--------------------------------------------------

kubectl port-forward svc/mlflow -n application 5000:5000

Відкрити у браузері:
http://localhost:5000


--------------------------------------------------
--------------------------------------------------
9. Перевірка результату
--------------------------------------------------

- ArgoCD UI показує Applications nginx та mlflow у стані Healthy / Synced
- У namespace application створені pod-и та сервіси
- Застосунки доступні через port-forward
- Git є єдиним джерелом правди (GitOps)

--------------------------------------------------
10. Прибирання ресурсів
--------------------------------------------------

Після перевірки завдання необхідно видалити ресурси, щоб уникнути витрат:

terraform destroy
