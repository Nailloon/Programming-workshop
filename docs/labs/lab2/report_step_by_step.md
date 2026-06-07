# Вторая лабораторная работа
[Ссылка на текст лабораторной работы](https://wiki.pmifi.ru/ru/disciplines/programming-workshop-masters-2sem)
Все последовательные действия и результат работы лежат рядом в папке images

## Предварительная установка что требуется поставить
1. Cкачать и поставить chocolatey для винды
2. Подготовка среды и установка Minikube + kubectl
3. Запустить PowerShell от имени администратора и вставить скрипт
```powershell
choco install kubernetes-cli
choco install minikube
```

## Начало работы с Minikube
1. В панели от имени администратора Powershell
```powershell
minikube start --driver=hyperv --memory=4096 --cpus=2 --disk-size=20000mb --cni=bridge
```

2.Затем идет настройка Ingress и того чтобы это нормально было проброшено для DNS
```powershell
minikube addons enable ingress
minikube addons enable ingress-dns
Get-DnsClientNrptRule | Where-Object {$_.Namespace -eq '.k8s'} | Remove-DnsClientNrptRule -Force
Add-DnsClientNrptRule -Namespace ".k8s" -NameServers "$(minikube ip)"
```

3. Проверка того что все запущено и работает
```powershell
minikube status
kubectl cluster-info
```

## Установка ArgoCD
1. Создать отдельное пространство имен:
```bash
kubectl create namespace argocd
```

2. Установить ArgoCD:
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

3. Дождитесь готовности всех подов:
```bash
kubectl wait --for=condition=Ready pods --all -n argocd
```

4. Переходим в VSCode со своим проектом и запускаем в консоли чтобы поднялся наш argocd
```bash
kubectl apply -f argocd-ingress.yaml
```

5. Проверка того что все правильно настроено и наш argocd работает + виден по DNS
```bash
kubectl get ingress -n argocd
nslookup argocd.cluster.k8s {minikube ip}
```

6. Чтобы окончательно проверить получаем пароль администратора для входа в UI:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

```powershell
 [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}")))
 ```

7. Открыть сайт и проверить что запускается и работает по данному домену Argocd + авторизоваться под admin с полученным паролем

https://argocd.cluster.k8s

## App of Apps + Guestbook
1. Запустить скрипт 
```bash
kubectl apply -f app.yaml
```
2. Сформировать папку для своего приложения как пример guestbook для example
3. Положить в эти папки конфигурации yaml для приложения
4. Сделать отдельно для этого сервиса yaml файлик в apps и правильно его настроить
5. Затем уже можно делать git push и синхронизировать через UI ArgoCD изменения
