# Вики Горпроектов

🚧🚧🚧
**Work in progress**
🚧🚧🚧

_**EN:** This repo contains setup instructions for a [MediaWiki](https://www.mediawiki.org/wiki/MediaWiki) instance that uses [Telegram login widget](https://core.telegram.org/widgets/login)_

## Ссылки по теме

### Телеграм-боты

[Bot Code Examples](https://core.telegram.org/bots/samples)

[Статья: Авторизация пользователей через Telegram](https://codex.so/telegram-auth)

## Разворачивание

### Необходимые условия

1.  Кластер на кубернетисе с настроенным подключением к нему

1.  Неймспейс `city4people-wiki`

    ```sh
    kubectl create namespace city4people-wiki
    ```

1.  Доступ к реестру с образами контейнеров

    См. <https://stackoverflow.com/a/61912590/1818285>

    ```sh
    GITHUB_USER=??
    GITHUB_TOKEN=??
    
    echo "{\"auths\":{\"docker.pkg.github.com\":{\"auth\":\"$(echo -n ${GITHUB_USER}:${GITHUB_TOKEN} | base64)\"}}}" | kubectl create secret generic dockerconfigjson-github-com --type=kubernetes.io/dockerconfigjson --from-file=.dockerconfigjson=/dev/stdin --namespace=city4people-wiki
    ```

### Телеграм-бот

```sh
TELEGRAM_BOT_TOKEN=??

kubectl create secret generic telegram-bot-token \
  --namespace=city4people-wiki \
  --from-literal=value=${TELEGRAM_BOT_TOKEN}

kubectl apply -f k8s/telegram-bot.yaml
```

### Вики-движок

- [README](https://hub.helm.sh/charts/bitnami/mediawiki)

- [values.yaml](https://github.com/bitnami/charts/blob/master/bitnami/mediawiki/values.yaml)

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami

MEDIAWIKI_PASSWORD=??
MARIADB_ROOTUSER_PASSWORD=??

cat <<EOF >/tmp/values.yaml
service:
  type: ClusterIP
persistence:
  enabled: false
allowEmptyPassword: no
mediawikiEmail: alexander@kachkaev.ru
mediawikiHost: city4people-wiki.ru
mediawikiName: Вики Горпроектов
mediawikiPassword: "${MEDIAWIKI_PASSWORD}"
mediawikiUser: admin
mariadb:
  rootUser:
    password: "${MARIADB_ROOTUSER_PASSWORD}"
EOF

## install
helm install --namespace=city4people-wiki mediawiki bitnami/mediawiki --values /tmp/values.yaml

## upgrade
helm upgrade mediawiki --namespace=city4people-wiki bitnami/mediawiki --values /tmp/values.yaml

## uninstall
helm uninstall --namespace=city4people-wiki mediawiki
```

```sh
kubectl apply -f k8s/mediawiki-ingress.yaml
```
