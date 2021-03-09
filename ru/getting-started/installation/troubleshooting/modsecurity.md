---
title: "Серверы с mod_security"
translation: "getting-started/installation/troubleshooting/modsecurity"
---

Этот документ охватывает довольно техническую тему, и любителям не рекомендуется пытаться это сделать. Новичкам в командной строке лучше всего доверить это профессиональному системному администратору или их хостинг-компании. Редактирование файлов конфигурации через командную строку может быть опасным, и вы можете разрушить свой сервер!

## ModSecurity (aka mod\_security или mod\_sec)

[ModSecurity](http://www.modsecurity.org/) - это брандмауэр веб-приложений с открытым исходным кодом, который работает как серверный модуль Apache. Он реализует исчерпывающий набор правил, реализующих усиление защиты общего назначения, и тем самым помогает исправлять распространенные проблемы безопасности веб-приложений. Он устанавливает внешний уровень безопасности, который повышает безопасность, обнаруживает и предотвращает атаки до того, как они достигнут веб-приложений. Он обычно доступен в системах cPanel как модуль EasyApache. Это хорошо зарекомендовавший себя модуль безопасности, который действительно может помочь защитить ваш сайт от распространенных векторов атак.

Мы подробно обсуждаем ModSecurity здесь, потому что менеджер MODX Revolution выдает много запросов, которые могут противоречить правилам `mod_security`.

### Тихий убийца

Менеджер MODX может просто спокойно выйти из строя, если одно из его действий заблокировано `mod_security`. Знай свой сервер! Проверьте журналы ошибок Apache! На карту поставлено ваше здравомыслие!

## Как я узнаю, что у меня установлен ModSecurity?

Прежде чем мы обсудим, как заставить ModSecurity и MODX хорошо работать вместе, вам нужно знать, установлено ли у вас это программное обеспечение на самом деле. Простое решение - спросить своего хостинг-провайдера, и, предположительно, они будут знать (если они не знают, какое программное обеспечение у них запущено, вероятно, пора [найти другую хостинговую компанию](https://modx.com/partners/hosting-saas/).

Если у вас есть собственный сервер (например, созданный на основе шаблона VPS), вы можете войти на сервер и проверить это самостоятельно.

### Проверка на сервере WHM

Многие VPS включают административные панели WHM/cPanel. Относительно легко проверить, запускаете ли вы `mod_security` на сервере WHM.

1. Войдите в свой экземпляр WHM (обычно по адресу <https://yoursite.com:2087/>).
2. Найдите раздел "Плагины" на левой панели навигации.
3. Если ModSecurity установлен, вы увидите **Mod Security** в списке ваших плагинов.

![](modsecurity+whm.jpg)

Удобный модуль cPanel/WHM `mod_security` доступен для визуального редактирования ваших правил здесь: <http://configserver.com/>

### Проверка через командную строку

Если у вас есть SSH-доступ к вашему серверу, вы можете проверить, какие модули Apache загружает при запуске. Чтобы распечатать, какие модули загружены в Apache, вы можете использовать утилиту **apachectl** в системах \*NIX, например

``` shell
apachectl -t -D DUMP_MODULES
```

Или, если ваша команда **apachectl** отсутствует в вашем текущем `$PATH`, вам может потребоваться указать полный путь к утилите. Чтобы найти путь, вы можете найти его с помощью команды **find**:

``` php
find / -name apachectl
```

Затем, когда вы найдете полный путь к утилите, вы можете выполнить команду подробно, например:

``` shell
/usr/local/apache/bin/apachectl -t -D DUMP_MODULES
```

Результат будет примерно таким:

``` php
Loaded Modules:
 core_module (static)
 rewrite_module (static)
 so_module (static)
 suphp_module (shared)
 security2_module (shared)  # <--- this is ModSecurity
```

Модуль `mod_security` указан как **security2\_module**

### Другой разведчик

Если у вас нет утилиты **apachectl** или вы не можете ее найти, вы можете вручную проверить файлы конфигурации Apache. Его можно настроить по-разному на разных серверах, но в целом Apache настроен на загрузку своего основного файла конфигурации, а затем при необходимости будет сканировать дополнительные каталоги на предмет дополнительных файлов конфигурации.

1. Проверьте основной файл Apache (часто находится тут **/etc/httpd**, например **/etc/httpd/conf/httpd.conf**)
2. Проверьте дополнительные каталоги конфигурации (часто внутри подпапок основного каталога конфигурации).

## Лог-файлы

После того, как вы убедились, что ModSecurity действительно запущен, вы захотите проверить свои журналы, чтобы увидеть, действительно ли ваши действия в диспетчере MODX вызывают срабатывание сигнализации системы безопасности. Лучше всего это сделать через командную строку: используйте SSH для входа на свой сервер и убедитесь, что у вас есть соответствующий доступ (например, привилегии root) для просмотра этих файлов журнала.

Основной журнал, за которым вы хотите следить, - это журнал ошибок Apache. Точное расположение настраивается в вашем файле конфигурации Apache, но часто оно находится внутри **/usr/local/apache/logs/error\_log**. Хороший способ просмотреть этот файл - использовать утилиту **tail**. Вы можете отслеживать файл в режиме реального времени, используя флаг **-f**, например

``` shell
tail -f /usr/local/apache/logs/error_log
```

Держите это окно открытым при навигации по диспетчеру MODX и будьте начеку, если в этом файле появятся какие-либо ошибки. (Нажмите ctrl-C, чтобы закрыть утилиту).

Вы также можете посмотреть содержимое журнала `mod_security`. Опять же, местоположение можно настроить, но часто оно сохраняется в **/usr/local/apache/logs/modsec\_audit.log**

### Пример ошибки

Если вы действительно видите, что ошибки регистрируются в журнале ошибок Apache, когда вы пытаетесь выполнить определенное действие в диспетчере MODX, велика вероятность, что ModSecurity просто помешал вам сделать что-то в диспетчере.

Вот пример ошибки из журнала ошибок Apache:

``` php
[Sat Nov 19 19:16:32 2011] [error] [client 123.123.123.123] ModSecurity: Access denied with code 500 (phase 2).
Pattern match "(insert[[:space:]]+into.+values|select.*from.+[a-z|A-Z|0-9]|select.+from|bulk[[:space:]]+insert|union.+select|convert.+\\\\(.*from)"
at ARGS:els.
[file "/usr/local/apache/conf/modsec2.user.conf"]
[line "359"]
[id "300016"]
[rev "2"]
[msg "Generic SQL injection protection"]
[severity "CRITICAL"]
[hostname "yoursite.com"]
[uri "/connectors/element/tv.php"]
[unique_id "TshG4EWntHMAAAfIFmUAAAAI"]
```

Из этой ошибки нам нужно 3 части информации, чтобы внести конкретное действие в белый список. Обратите внимание на следующие 3 пункта:

``` php
[id "300016"]
[hostname "yoursite.com"]
[uri "/connectors/element/tv.php"]
```

Это говорит о том, какое правило сработало, в каком домене оно сработало и из какого места внутри этого домена сработало правило.

## Внесение правила для домена в белый список

Внесение правила в белый список для определенного домена осуществляется с помощью файла "включаемых". Это требует нескольких шагов, чтобы сделать это безопасно.

### Восстановите конфигурацию Apache

Первое, что нужно сделать, - это создать резервную копию и перестроить файл httpd.conf, чтобы убедиться в отсутствии ошибок (выполните следующие команды по очереди)

``` shell
cd /usr/local/apache/conf
cp -p httpd.conf httpd.conf.backup
```

Если вы используете сервер cPanel, вы можете перестроить файл, выполнив следующую команду:

``` php
/scripts/rebuildhttpdconf
```

Цель здесь - убедиться, что ваш существующий файл конфигурации Apache зарезервирован и работает _перед тем_, как мы попытаемся изменить его.

### Отредактируйте файл виртуальных хостов

Многие настройки (включая настройки cPanel) не хотят, чтобы вы возились напрямую с основным файлом конфигурации Apache. Вместо этого вы отредактируете файл vhosts для данного домена. Просмотрите свой основной файл конфигурации Apache (**httpd.conf**) и найдите свое доменное имя, чтобы узнать, куда он передал свои файлы конфигурации. Вы должны найти некоторые ссылки на него внутри блока **VirtualHost**.

``` php
<VirtualHost 123.123.123.123>
    ServerName yoursite.com
    ServerAlias www.yoursite.com
    DocumentRoot /home/youruser/public_html
    # ... more stuff here ...
    Include "/usr/local/apache/conf/userdata/std/2/yoursite/*.conf"
    Include "/usr/local/apache/conf/userdata/std/2/yoursite/yoursite.com/*.conf"
</VirtualHost>
```

Основываясь на этой директиве **VirtualHosts**, мы можем обратить внимание на 2 указанных каталога:

- /usr/local/apache/conf/userdata/std/2/yoursite/
- /usr/local/apache/conf/userdata/std/2/yoursite/yoursite.com/

Вы также можете установить общие серверные правила в файле:

- /usr/local/apache/conf/modsec2/whitelist.conf

Вот где Apache будет искать дополнительные конфигурации. Если вы знаете, что вам не нужно беспокоиться о дополнительных файлах конфигурации, вы можете перейти к следующему разделу и просто добавить свои правила белого списка. Если вы работаете на сервере cPanel или используете какой-либо другой тип настройки, при котором вы либо не можете, либо не должны редактировать основной файл **httpd.conf** напрямую, тогда вам следует поместить свои правила в отдельный файл конфигурации. Возможно, вам потребуется создать каталоги, перечисленные выше, или, возможно, вам придется немного выполнить rtfm, чтобы выяснить, где Apache будет искать дополнительные файлы конфигурации.

## Добавить правило белого списка

### Общий пример

Общее правило белого списка выглядит так:

``` php
<LocationMatch "/path/to/URI">
  <IfModule mod_security2.c>
    SecRuleRemoveById (Rule number)
    SecRuleRemoveById (Rule number, if more for this domain)
    SecRuleRemoveById (etc)
    SecRuleRemoveById (etc)
  </IfModule>
</LocationMatch>
```

Вы можете изменить его и добавить в свою директиву VirtualHosts (либо в вашем основном **httpd.conf**, либо во внешних **vhosts.conf** файлах). Пока Apache загружает файл конфигурации, правило белого списка будет зарегистрировано.

### Конкретный пример

Приведите наш пример сообщения об ошибке ранее, в котором указана следующая ошибка:

``` php
[id "300016"]
[hostname "yoursite.com"]
[uri "/connectors/element/tv.php"]
```

Мы могли бы перейти к директиве VirtualHosts для **yoursite.com** и добавить следующее правило:

``` php
<LocationMatch "/connectors/element/tv.php">
  <IfModule mod_security2.c>
    SecRuleRemoveById 300016
  </IfModule>
</LocationMatch>
```

Обратите внимание, что он ссылается на коннектор MODX по его пути и ссылается на правило ModSecurity по его идентификатору.

## Остерегайтесь перемещения вашего сайта

Если вы переместите свой сайт в новый каталог или каталог ** коннекторов ** в нестандартное место, вам придется отредактировать свои правила! Они применяются к определенному URL-адресу, поэтому, если ваши URL-адреса изменятся, правила придется обновить.

### Более широкий пример

Может быть неприятно просматривать функциональность MODX по одному экрану администратора за раз, но, похоже, есть некоторые трудности с занесением в белый список целых каталогов. Подумайте о переименовании вашего каталога «коннекторов» (см. [Укрепление MODX Revolution](administering-your-site/security/hardening-modx-revolutio)).

``` php
<LocationMatch "/manager/index.php">
SecRuleRemoveById 300016
</LocationMatch>

<LocationMatch "/connectors/resource/index.php">
  SecRuleRemoveById 300013 300014 300015 300016
</LocationMatch>

<LocationMatch "/connectors/element/tv.php">
  SecRuleRemoveById 300013 300016
</LocationMatch>
```

## Перезагрузите Apache

После внесения изменений в файлы конфигурации вам необходимо перезапустить Apache, чтобы конфигурации можно было перечитать.

### cPanel: перестроить файл Conf

Если вы не используете сервер cPanel, пропустите этот шаг и просто перезапустите Apache.

На сервере cPanel вы захотите повторно запустить утилиту **rebuildhttpdconf**:

``` shell
cd /usr/local/apache/conf
/scripts/rebuildhttpdconf
```

Затем вы можете проверить, внесены ли изменения, внесенные вами во внешние файлы, в основной файл (опять же, это ТОЛЬКО при настройке cPanel: cPanel вносит внешние конфигурации в основной файл **httpd.conf**). Попробуйте просмотреть файл, чтобы увидеть, что все, что вы поместили во внешний файл, теперь включено в основной файл.

### Перезагрузите Apache

После внесения изменений перезапустите процесс Apache:

``` shell
/etc/init.d/httpd restart
```

Если в ваших файлах есть ошибки, вы будете предупреждены о них. Это может нервировать, потому что, если Apache не вернется в сеть, **ваш сайт будет недоступен!**

## Статические ресурсы

ModSecurity может повлиять на ваши статические ресурсы MODX (или любой PHP-скрипт, который читает файл для загрузки пользователем). Что может случиться, так это то, что если ваш файл слишком большой, загрузка будет преждевременно прервана, и вы получите поврежденный файл. Часто размер загруженного файла составляет всего около 64 КБ, даже если исходный файл может быть значительно больше. Если вы столкнулись с этим, это может быть хорошим намеком на то, что ModSecurity вмешивается. **Для этого может не быть записи в журнале** (!!!), поэтому может быть очень сложно отследить это поведение обратно в ModSecurity!

В WHM вы можете редактировать параметры конфигурации ModSecurity, щелкнув ссылку плагина «Mod Security» (изображенную ранее на этой странице) и нажав кнопку «Изменить конфигурацию».

Детали конфигурации, которые могут повлиять на ваши загрузки, следующие:

- SecRequestBodyAccess
- SecRequestBodyLimit
- SecRequestBodyInMemoryLimit

Простое решение - полностью обойти ModSecurity для таких загрузок:

``` php
SecRequestBodyAccess Off
```

См. <Http://www.modsecurity.org/documentation/modsecurity-apache/2.1.0/modsecurity2-apache-reference.html> для получения дополнительной информации о различных деталях конфигурации.

Другой причиной этого загадочного симптома может быть конфликт между веб-серверами: например, если у вас установлены Apache и NGINX на одном сервере, _ убедитесь, что они оба не используют сжатие gzip_ - результат может быть очень похож на вмешательство ModSecurity! Если NGINX сжимает большой статический ресурс, а затем Apache также пытается его сжать, это не удается, и файл заканчивается обрезанием до 64 КБ.

## Смотрите также

[Справочник по настройке ModSecurity](http://www.modsecurity.org/documentation/modsecurity-apache/2.1.0/modsecurity2-apache-reference.html)

1. [Lighttpd Руководство](getting-started/friendly-urls/lighttpd)
2. [Установка на сервере под управлением ModSecurity](getting-started/installation/troubleshooting/modsecurity)
3. [Конфигурация сервера Nginx](getting-started/friendly-urls/nginx)