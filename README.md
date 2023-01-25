# Telegram auth extensions for MediaWiki

> **EN:** This repo contains setup instructions for a [MediaWiki](https://www.mediawiki.org/wiki/MediaWiki) instance that uses [Telegram auth](https://core.telegram.org/widgets/login) for user to login
>
> 👀 [English version via Google Translate](https://translate.google.com/translate?sl=ru&tl=en&u=https://github.com/grosvold/MediaWiki-TelegramAuth
Public/blob/main/README.md)

## Цель проекта:

1. Вырезать все лишнее из проекта [city4people-wiki](https://github.com/kachkaev/city4people-wiki)
2. Обновить TelegramAuth расширение для актуальных версий PluggableAuth 6.2 и MediaWiki 1.39



Моя личная цель - организовать авторизацию на 
[Вики](https://ru.wikipedia.org/wiki/Вики) проекта [wiki.euc.club](https://wiki.euc.club) посредством [Telegram Login Widget](https://core.telegram.org/widgets/login)


TelegramAuth  
* version: 1.0

requires:  
* MediaWiki: >= 1.34.0,  
* extensions: PluggableAuth: >= 5.1

author:  
* [Kachkaev (github)](https://github.com/kachkaev/)  
* [Kachkaev (MediaWiki)](https://www.mediawiki.org/wiki/User:Kachkaev)  
* [Banakin (MediaWiki)](https://www.mediawiki.org/wiki/User:Banakin900)

[Адрес расширения на MediaWiki](https://www.mediawiki.org/wiki/Extension:TelegramAuth) (удалено или не было опубликовано)

## Что не так?
* Текущая реализация бота не работает с актуальной PluggableAuth 6.2 [REL1_39-e7de886](https://extdist.wmflabs.org/dist/extensions/PluggableAuth-REL1_39-e7de886.tar.gz), при включении TelegramAuth выдаётся ошибка типа  
[a2c074fed04df0fc5e75ecf1] 2023-01-22 02:49:40: Неустранимое исключение типа «Error».  
* с версией PluggableAuth 5.7 [REL1_35-6d28813](https://extdist.wmflabs.org/dist/extensions/PluggableAuth-REL1_35-6d28813.tar.gz), если я правильно понял авторизацию PluggableAuth принимает, переадресует на пустую страницу (https://wiki.euc.club/index.php/Служебная:PluggableAuthLogin) но не может её передать в MediaWiki. При переходе на главную страницу видно, что авторизация не прошла.  
* PluggableAuth 5.7 [REL1_37-c1cc644](https://extdist.wmflabs.org/dist/extensions/PluggableAuth-REL1_37-c1cc644.tar.gz) также как с REL1_35-6d28813  
* PluggableAuth 5.7 [REL1_36-17859c9](https://extdist.wmflabs.org/dist/extensions/PluggableAuth-REL1_36-17859c9.tar.gz) работа плагина с MediaWiki 1.39.0 не проверялась.  
* PluggableAuth 6.1 [REL1_38-d7cb5c7](https://extdist.wmflabs.org/dist/extensions/PluggableAuth-REL1_38-d7cb5c7.tar.gz) работа плагина с MediaWiki 1.39.0 не проверялась.

## Лицензия
Автор [Banakin (MediaWiki)](https://www.mediawiki.org/wiki/User:Banakin900) удалил свое детище с github, однако [Kachkaev (github)](https://github.com/kachkaev/) одобрил использование под лицензией [BSD-3-Clause](https://opensource.org/licenses/BSD-3-Clause)

---
## Ниже частично прореженные инструкции от kachkaev 
(Нужно чистить)

[Медиавики](https://mediawiki.org) — тот же движок, что и Википедия.
Этот выбор определяет низкий порог входа для участников, а также задаёт понятные и проверенные правила для взаимодействия в сообществе.

Все технические компоненты вики имеют открытый исходный код и не требуют платных лицензий.
Если обслуживание ресурса осуществляется волонтёрами, платным остаётся только веб-хостинг.

Движок настроен так, чтобы процесс редактирования содержимого мало чем отличался от Википедии.
Единственная существенная разница в настройке Медиавики — [авторизация через Телеграм](https://core.telegram.org/widgets/login).
Благодаря этой интеграции участникам не приходится придумывать логины и пароли, а администраторам проще отслеживать и модерировать правки.

Войти в вики может любой пользователь Телеграма при наличии имени пользователя в профиле.
Существует техническая возможность варьировать доступ к страницам в зависимости от статуса человека — это требует дополнительной настройки.

---


### Развёртывание Телеграм-бота

[Авторизация через Телеграм](https://core.telegram.org/widgets/login) требует наличия бота.
Этот бот должен быть привязан к домену сервера, на котором размещена кнопка входа.

Для создания бота следует отправить команду `/newbot` боту [@BotFather](https://t.me/BotFather).
Привязка бота к домену настраивается там же.

Развёртывать сервер для бота строго говоря не обязательно, так как весь процесс авторизации проводит сам Телеграм.
Работающий бот лишь отвечает на сообщения пользователей ссылкой на вики.



#### Синхронизация ресурсов Медиавики из этого репозитория

Хранилище пода с Медиавики содержит файл `mediawiki/LocalSettings.php`.
После создания _нового_ экземпляра веб-приложения следует вручну добавить в него эти две строчки:

```php
// в начале файла
import "./LocalSettingsDebug.php";

// в конце файла
`import "./LocalSettingsTailoring.php";`
```

Команды для синхронизации ресурсов из локальной копии этого репозитория:

```sh
## модуль для авторизации через Телеграм
rsync --archive --delete --stats --human-readable custom-mediawiki-resources/extensions/TelegramAuth/ ${MEDIAWIKI_PV_SSH_HOST}:${MEDIAWIKI_PV_PATH}/mediawiki/extensions/TelegramAuth
```

```sh
## Донастройка движка (LocalSettingsDebug.php и LocalSettingsTailoring.php)
rsync --archive --stats --human-readable custom-mediawiki-resources/LocalSettings*.php ${MEDIAWIKI_PV_SSH_HOST}:${MEDIAWIKI_PV_PATH}/mediawiki/
```

```sh
## Логотипы
rsync --archive --stats --human-readable visuals/main/mediawiki/*.png ${MEDIAWIKI_PV_SSH_HOST}:${MEDIAWIKI_PV_PATH}/mediawiki/images/
```

#### Группа `ssotelegram`

Участникам, вошедшим через Телеграм, автоматически присваивается группа `ssotelegram` (_SSO – Single sign-on_).
Это может упростить контроль [прав доступа](https://www.mediawiki.org/wiki/Manual:User_rights) в будущем.

Чтобы группа `ssotelegram` отображалась на страницах вики по-русски, следует создать три служебные страницы:

- `MediaWiki:Group-ssotelegram`

  > Пользователи Телеграма

- `MediaWiki:Group-ssotelegram-member`

  > пользователь Телеграма

- `MediaWiki:Grouppage-ssotelegram`
  > Пользователи Телеграма

#### Расширения

Установка дополнительных расширений для Медиавики.

Эти же команды используются для обновления расширений после выхода новых версий.

```sh
## ℹ️ Терминал внутри MEDIAWIKI_PV_SSH_HOST

EXTENSIONS_DIR=${MEDIAWIKI_PV_PATH}/mediawiki/extensions

mv ${EXTENSIONS_DIR}/PluggableAuth ${EXTENSIONS_DIR}/PluggableAuth.bak
wget -c https://extdist.wmflabs.org/dist/extensions/PluggableAuth-REL1_35-2a465ae.tar.gz -O - | tar -xz -C $EXTENSIONS_DIR

chmod 755 ${EXTENSIONS_DIR}/Scribunto/includes/engines/LuaStandalone/binaries/lua5_1_5_linux_64_generic/lua

# rm -rf ${EXTENSIONS_DIR}/*.bak
```

#### Темы оформления

```sh
## ℹ️ Терминал внутри MEDIAWIKI_PV_SSH_HOST

SKINS_DIR=${MEDIAWIKI_PV_PATH}/mediawiki/skins

# https://www.mediawiki.org/wiki/Skin:Minerva_Neue
mv ${SKINS_DIR}/MinervaNeue ${SKINS_DIR}/MinervaNeue.bak
wget -c https://extdist.wmflabs.org/dist/skins/MinervaNeue-REL1_35-bb52d27.tar.gz -O - | tar -xz -C $SKINS_DIR

# rm -rf ${SKINS_DIR}/*.bak
```

#### Шаблоны и стили

После создания нового экземпляра вики следует импортировать шаблоны и стили из русской Википедии.
Действия ниже можно периодически повторять для обновления.

1.  Зайти на на страницу <https://ru.wikipedia.org/wiki/Служебная:Экспорт>

    1.  Вбить в форму экспорта следующий текст:

        ```txt
        MediaWiki:Common.css
        MediaWiki:Minerva.css
        MediaWiki:Mobile.css
        Шаблон:Ambox
        Шаблон:Внимание
        ```

    1.  Поставить все три галочки

    1.  Нажать «Экспортировать» и сохранить файл локально

1.  Зайти на страницу `Служебная:Импорт` нужного экземпляра вики

    1.  Выбрать скаченный файл `xml`

    1.  В качестве префикса интервики выбрать `wikipedia_ru`

    1.  Нажать «Загрузить файл»

1.  Закомменировать стили главной страницы в `Mobile.css`, чтобы убрать серый фон

#### Настройка расширения для авторизации через Телеграм

Настройки расширения `TelegramAuth` включены в файл `LocalSettingsTailoring.php`.
В случае изменения токена бота, его хэш придётся перегенерировать.
Поможет эта команда:

```sh
# export TELEGRAM_BOT_TOKEN=
node --eval 'console.log(require("crypto").createHash("sha256").update(process.env.TELEGRAM_BOT_TOKEN).digest("hex"));'
```

### Настройка резервного копирования

Резервные копии данных Медивики создаются автоматически раз в сутки хостинг-провайдером.

> TODO: Настроить отправку копии на _AWS S3_ или другие облачные хранилища при помощи Кубернетиса

### Хаки для песочницы

Чтобы песочница визуально отличалась от основной вики, следует добавить следующий код в начало двух страниц `MediaWiki:*.css` — фон станет жёлтым.

`MediaWiki:Common.css` (настольный режим)

```css
/* begin sandbox hacks */

body {
  background-color: #ffeeb2 !important;
}

#mw-page-base {
  background-image: linear-gradient(
    to bottom,
    #ffffff 50%,
    #ffeeb2 100%
  ) !important;
}

.vector-menu-tabs li {
  background-image: linear-gradient(
    to top,
    #77c1f6 0,
    #ffeeb2 1px,
    #ffffff 100%
  ) !important;
}

.vector-menu-tabs .selected {
  background: #ffffff !important;
}

/* end sandbox hacks */
```

`MediaWiki:Mobile.css` (мобильный режим)

```css
/* begin sandbox hacks */

#mw-mf-page-center,
#mw-mf-page-left,
.header-container.header-chrome {
  background-color: #ffeeb2 !important;
}

/* end sandbox hacks */
```

## Заметки по Медиавики

### Полезные служебные страницы

- `MediaWiki:Sitenotice`
- `Special:Version`
- `Template:Infobox`

### Отладка шаблонов

`&uselang=qqx`

<!--

## TODO
### шаблоны

stub
уточнить
infobox

### TODO

robots.txt
social preview

-->
