# goit-argo
Lesson 7 – GitOps with ArgoCD and Helm (MLflow)

Мета цього завдання — реалізувати GitOps-підхід у Kubernetes за допомогою ArgoCD. У межах роботи ArgoCD було розгорнуто в існуючому кластері AWS EKS через Terraform як Helm-реліз. Далі створено окремий Git-репозиторій з описом деплою застосунку (MLflow / test application), який ArgoCD автоматично підхоплює та синхронізує з кластером.

========================
1. Розгортання ArgoCD через Terraform
========================

ArgoCD розгортається у namespace infra-tools як Helm release за допомогою Terraform. Усі значення Helm-чарту винесені в окремий файл argocd-values.yaml.

Структура Terraform-проєкту:
terraform/argocd
├── main.tf
├── provider.tf
├── variables.tf
├── outputs.tf
├── backend.tf
├── terraform.tf
└── values
    └── argocd-values.yaml

Запуск Terraform:

cd terraform/argocd
terraform init
terraform apply

Перевірка, що ArgoCD успішно запущений:

kubectl get pods -n infra-tools

У namespace infra-tools мають бути pod-и з префіксом argocd-.

========================
2. Доступ до ArgoCD UI
========================

Для доступу до веб-інтерфейсу ArgoCD використовується port-forward (команда блокує термінал, тому виконується в окремій вкладці):

kubectl port-forward svc/argocd-server -n infra-tools 8080:443

Після цього ArgoCD UI доступний за адресою:

https://localhost:8080

Логін: admin

Отримання початкового пароля:

kubectl get secret argocd-initial-admin-secret -n infra-tools -o jsonpath="{.data.password}" | base64 --decode

========================
3. Git-репозиторій для GitOps
========================

Створено окремий Git-репозиторій для GitOps-деплою:

https://github.com/OlesiaKh/goit-argo

Усі зміни виконуються в гілці lesson-7.

Структура репозиторію:

goit-argo
├── namespaces
│   ├── application
│   │   ├── ns.yaml
│   │   └── nginx.yaml
│   └── infra-tools
│       └── ns.yaml
├── application.yaml
└── README.md

У репозиторії описані:
- namespace-и
- маніфести застосунку
- ArgoCD Application

========================
4. ArgoCD Application (Helm / Git)
========================

У файлі application.yaml описано ArgoCD Application, який:

- використовує Git-репозиторій як джерело
- автоматично синхронізується (auto-sync)
- має self-heal
- автоматично створює namespace
- деплоїть застосунок у namespace application

Застосування Application у кластері:

kubectl apply -f application.yaml

Перевірка статусу Application:

kubectl get applications -n infra-tools

Очікуваний статус: Synced, Healthy.

========================
5. Автоматичний деплой застосунку (MLflow / test app)
========================

Після git push ArgoCD автоматично:

- підхоплює зміни з Git
- створює Deployment, Service та Pod-и
- підтримує стан кластера відповідно до Git (GitOps)

Перевірка pod-ів:

kubectl get pods -n application

Перевірка сервісів:

kubectl get svc -n application

========================
6. Доступ до сервісу
========================

Для доступу до сервісу використовується port-forward:

kubectl port-forward svc/nginx -n application 8081:80

Після цього сервіс доступний у браузері:

http://localhost:8081

(Для MLflow доступ організовується аналогічно через відповідний Service.)

========================
