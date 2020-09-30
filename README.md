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

```sh
export TELEGRAM_BOT_TOKEN=??
node --eval 'console.log(require("crypto").createHash("sha256").update(process.env.TELEGRAM_BOT_TOKEN).digest("hex"));'
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
  master:
    master:
      persistence:
        size: 10Gi
  rootUser:
    password: "${MARIADB_ROOTUSER_PASSWORD}"
persistence:
  size: 20Gi
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

### Шаги после создания сервера

### Заметки

Extensions:

- <https://www.mediawiki.org/wiki/Extension:Echo>
- <https://www.mediawiki.org/wiki/Extension:MobileFrontend>
- <https://www.mediawiki.org/wiki/Extension:StructuredDiscussions>
- <https://www.mediawiki.org/wiki/Extension:CirrusSearch>

Default user options: <https://www.mediawiki.org/wiki/Manual:$wgDefaultUserOptions>
`/opt/bitnami/mediawiki`

```php
// Debugging
error_reporting(-1);
ini_set('display_startup_errors', 1);
ini_set('display_errors', 1);

$wgShowExceptionDetails = true;
$wgDebugToolbar = true;
// $wgShowDebug=true;
$wgDevelopmentWarnings = true;
$wgDebugDumpSql = true;
$wgDebugLogFile = dirname(__FILE__) . "/debug.log";
$wgDebugComments = true;
$wgEnableParserCache = false;
$wgCachePages = false;
```

```wiki
<p style="text-align:left">
🚨🚨🚨<br>
'''Сайт в процессе настройки и пока ещё не готов к наполнению'''<br>
🚨🚨🚨
</p>

<p style="text-align:left">
Экспериментальная вики [https://city4people.ru Горпроектов] создаётся для координации работы в Пензенском отделении. Если всё получится, область применения расширится до федерального уровня. Вики Горпроектов — частная инициатива. По любым вопросам пишите в телеграм [https://t.me/kachkaev @kachkaev]
</p>
```

## Auth

List of login extensions: <https://www.mediawiki.org/wiki/Category:Login_extensions>

## Other extensions

<https://meta.miraheze.org/wiki/Special:Version>

## Notable Special pages

- Template:Infobox
- MediaWiki:Sitenotice

## Template debugging

`&uselang=qqx`

```sh
ssh remote mkdir -p /top/a/b/c/

rsync --archive --delete --stats --human-readable TelegramAuth/ kachkaev--firstvds--city4people-wiki:/var/www/mediawiki-main/mediawiki/extensions/TelegramAuth
```
