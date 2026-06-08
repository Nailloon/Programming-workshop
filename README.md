# GitOps с ArgoCD

## Содержание

- [Предыстория](#предыстория)
- [Установка](#установка)
- [Запуск ЛР2](#запуск-лр2)
- [Запуск ЛР3](#запуск-лр3)

## Предыстория

**Сервис позволяет:**

- совместно редактировать структурированные конспекты
- легко дополнять и экспортировать лекции (например, в PDF)
- хранить и структурировать знания по курсам/темам
- работать с изображениями и вложениями
- предоставлять публичные ссылки и обеспечивать SSO‑вход

**ЛР1** — анализ требований к системе для совместного конспектирования лекций (Markdown/LaTeX, live-коллаборация, SSO, экспорт, self-host). Выбран **Outline**.

**ЛР2** — развёртывание Minikube, настройка Ingress, установка ArgoCD, реализация паттерна **App of Apps** с демо-приложением Guestbook.

**ЛР3** — развёртывание Outline поверх инфраструктуры из ЛР2, интеграция с Dex (OIDC).

## Установка

```powershell
# Установка Chocolatey (если ещё не установлен)
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Установка Minikube и kubectl
choco install kubernetes-cli minikube git -y

# Запуск Minikube с драйвером Hyper-V
minikube start --driver=hyperv --memory=4096 --cpus=2 --disk-size=20000mb --cni=bridge
```

### Проверка установки

```powershell
minikube status
kubectl get nodes
```

## Запуск ЛР2

```powershell
# 1. Настроить Ingress и DNS
minikube addons enable ingress
minikube addons enable ingress-dns
Add-DnsClientNrptRule -Namespace ".k8s" -NameServers "$(minikube ip)"

# 2. Установить ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 3. Дождаться готовности подов
kubectl wait --for=condition=Ready pods --all -n argocd

# 4. Применить Ingress для ArgoCD
kubectl apply -f argocd-ingress.yaml

# 5. Запустить App of Apps
kubectl apply -f app.yaml

# 6. Получить пароль admin
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}")))

# 7. Открыть в браузере
# https://argocd.cluster.k8s
```

## Запуск ЛР3
```powershell
# 1. Убедиться, что инфраструктура ЛР2 работает
minikube status
kubectl get nodes

# 2. Синхронизировать приложения через ArgoCD UI
# Перейти в https://argocd.cluster.k8s
# Нажать "Sync" для приложений dex и outline

# 3. Проверить, что поды запустились
kubectl get pods -n auth
kubectl get pods -n outline

# 4. Добавить DNS для Outline (если ещё не добавлен)
Add-DnsClientNrptRule -Namespace ".k8s" -NameServers "$(minikube ip)"

# 5. Открыть в браузере
# https://outline.cluster.k8s

# 6. Войти через Dex

# 7. Создать тестовый документ
# Вставить формулу: $$ E = mc^2 $$
# Прикрепить изображение
```