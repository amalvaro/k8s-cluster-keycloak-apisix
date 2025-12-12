# k8s-cluster-keycloak-apisix

Инфраструктура для локального развёртывания Apache APISIX (с дашбордом), Keycloak (с консолью администрирования) и NGINX Ingress Controller в кластере Kind.

## Подготовка
1. Установите `kind`, `kubectl` и `helm`.
2. Добавьте записи в `/etc/hosts` для доступа к дашбордам через Ingress:
   ```
   127.0.0.1 apisix.local
   127.0.0.1 keycloak.local
   ```

## Создание кластера Kind
```bash
kind create cluster --config k8s/kind/cluster.yaml
```

## Установка NGINX Ingress Controller
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  -f k8s/ingress-nginx/values.yaml
```

## Установка Apache APISIX
```bash
helm repo add apisix https://charts.apiseven.com
helm repo update
helm upgrade --install apisix apisix/apisix \
  --namespace apisix --create-namespace \
  -f k8s/apisix/values.yaml
```

### Установка APISIX Dashboard
```bash
helm upgrade --install apisix-dashboard apisix/apisix-dashboard \
  --namespace apisix \
  -f k8s/apisix-dashboard/values.yaml
```

## Установка Keycloak
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm upgrade --install keycloak bitnami/keycloak \
  --namespace keycloak --create-namespace \
  -f k8s/keycloak/values.yaml
```

## Настройка Ingress для дашбордов
Примените общие Ingress-манифесты после установки всех чартов:
```bash
kubectl apply -f k8s/ingress/app-ingresses.yaml
```

## Доступ
- APISIX Dashboard: http://apisix.local
  - Логин/пароль: `admin` / `admin123`
- Keycloak: http://keycloak.local
  - Администратор: `admin` / `admin123`

Порты 80 и 443 проброшены на хост через конфигурацию Kind. Дополнительно проброшены порты 9080/9443 (шлюз APISIX) и 9000 (дашборд) на случай прямого обращения к сервисам NodePort.
