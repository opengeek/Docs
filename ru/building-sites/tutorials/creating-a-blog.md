---
title: Создание блога
translation: case-studies-and-tutorials/creating-a-blog-in-modx-revolution
---

## Требования:

1. Требования расширения могут требовать использования FURL, и ".html" можно изменить на "/": Content -> Types -> HTML (.html) -> /

## Создание блога в MODX Revolution

Это руководство поможет вам настроить гибкое и мощное решение для ведения блогов в MODX Revolution. Поскольку MODX Revolution - это не программное обеспечение для ведения блогов, а полноценная платформа для приложений контента, он не поставляется с готовым решением для ведения блогов. Вам нужно будет настроить свой блог так, как вы этого хотите.

К счастью, инструменты для этого уже есть для вас. Из этого туториала вы узнаете, как их настроить. Рекомендуется ознакомиться с Revolution [синтаксисом тегов](building-sites/tag-syntax "Tag Syntax"), прежде чем мы начнем.

Однако, прежде чем мы начнем, это обширное руководство, которое покажет вам, как создать мощный блог с публикацией, архивированием, добавлением тегов, комментариями и многим другим. Если вам не нужна какая-то конкретная часть, просто пропустите эту часть. MODx является модульным, и ваш блог может функционировать в любом объеме, который вам нравится. И, опять же, это только один способ сделать это - существует множество способов создать блог в MODx Revolution.

Этот учебник был основан на настройке блога на [splittingred.com](http://splittingred.com/). Если вы хотите получить демонстрацию перед чтением, посмострите там.

## Получение необходимых дополнений

Прежде всего, вы захотите скачать и установить некоторые дополнения, которые мы будем использовать в нашем блоге. Ниже приведен список наиболее часто используемых дополнений:

### Необходимые дополнения

- [getResources](/extras/getresources "getResources") - Для публикации сообщений, страниц и других ресурсов.
- [getPage](/extras/getpage "getPage") - Для разбиения списков на страницы.
- [Quip](/extras/quip "Quip") - Все и вся в комментировании.
- [tagger](/extras/tagger "tagger") - Для управления тегами и выполнения навигации на основе тегов.
- [Archivist](/extras/archivist "Archivist") - Для управления вашим разделом архивов.

### Дополнительные дополения

- [Breadcrumbs](/extras/breadcrumbs "Breadcrumbs") - Для отображения хлебных крошек
- [Gallery](/extras/gallery "Gallery") - Для управления фото галереями.
- [SimpleSearch](/extras/simplesearch "SimpleSearch") - Для добавления простого поиска на ваш сайт.
- [getFeed](/extras/getfeed "getFeed") - Если вы хотите получить другие каналы на своем сайте, например, канал Twitter.
- [Login](/extras/login "Login") - Если вы хотите ограничить комментирование только зарегистрированными пользователями, это вам понадобится.

## Создание шаблона блога

Во-первых, вы хотите иметь шаблон, предназначенный только для сообщений в блоге. Зачем? Что ж, если вам нужны комментарии и специальное форматирование или отображение страниц для вашего блога, вы, вероятно, не захотите делать это для каждой записи блога. Итак, лучший способ - это создать собственный шаблон блога. В этом руководстве уже предполагается, что у вас есть базовый шаблон для ваших обычных страниц на сайте - позже мы будем ссылаться на него как «BaseTemplate».

Мы создадим один с именем 'BlogPostTemplate' Наш контент выглядит примерно так:

```php
[[$pageHeader]]
<div id="content" class="blog-post">
 <h2 class="title"><a href="[[~[[*id]]]]">[[*pagetitle]]</a></h2>
 <p class="post-info">
 Posted on [[*publishedon:strtotime:date=`%b %d, %Y`]] |
 Tags: [[*tags:notempty=`[[!tolinks? &items=`[[*tags]]` &tagKey=`tag` &target=`1`]]`]] |
 <a href="[[~[[*id]]]]#comments" class="comments">
   Comments ([[!QuipCount? &thread=`blog-post-[[*id]]`]])
 </a>
 </p>
 <div class="entry">
   <p>[[*introtext]]</p>
   <hr />
   [[*content]]
  </div>
 <div class="postmeta">
   [[*tags:notempty=`
<span class="tags">Tags: [[!tolinks? &items=`[[*tags]]` &tagKey=`tag` &target=`1`]]</span>
   `]]
   <br class="clear" />
 </div>
 <hr />
 <div class="post-comments" id="comments">[[!Quip?
     &thread=`blog-post-[[*id]]`
     &replyResourceId=`123`
     &closeAfter=`30`
   ]]
   <br /><br />
   [[!QuipReply?
      &thread=`blog-post-[[*id]]`
      ¬ifyEmails=`my@email.com`
      &moderate=`1`
      &moderatorGroup=`Moderators`
      &closeAfter=`30`
   ]]
 </div>
</div>
[[$pageFooter]]
```

Итак, давайте рассмотрим шаблон, не так ли? По ходу дела помните об этом - вы можете перемещать любые из этих частей, изменять их параметры и корректировать их размещение. Это исключительно базовая структура - например, если вы хотите, чтобы ваши теги были внизу, переместите их туда! MODX не ограничивает вас в этом.

### Хедер и фуутер

Во-первых, вы заметите, что у меня есть два блока: «pageHeader» и «pageFooter». Эти блоки содержат мои общие HTML-теги, которые я бы поместил в нижний колонтитул и заголовок на своем сайте, чтобы я мог использовать их в разных шаблонах. Полезно, если я не хочу обновлять какие-либо изменения верхнего / нижнего колонтитула в каждом из моих шаблонов - я могу просто сделать это за один фрагмент. После этого я помещаю заголовок страницы своего ресурса и делаю его ссылкой, которая ведет вас на ту же страницу.

### Информация о посте

Далее мы попадаем в «информацию» поста - в основном это информация об автора и теги для поста. Давайте посмотрим подробно:

```php
<p class="post-info">
Posted on [[*publishedon:strtotime:date=`%b %d, %Y`]] |
Tags: [[*tags:notempty=`[[!tolinks? &items=`[[*tags]]` &tagKey=`tag` &target=`1`]]`]] |
<a href="[[~[[*id]]]]#comments" class="comments">
 Comments ([[!QuipCount? &thread=`blog-post-[[*id]]`]])
</a>
</p>
```

Первая часть берет поле publishedon с Ресурса и форматирует его в красивую дату.

Вторая часть:  мы отображаем список тегов для этой записи блога. Вы можете увидеть, как мы ссылаемся на переменную шаблона «tags» - мы еще не создали ее, так что не беспокойтесь - и затем передайте ее как свойство фрагменту tolinks. Снипет tolinks поставляется с [tagLister](/extras/taglister "tagLister"), и переводит теги с разделителями в ссылки. Это означает, что наши теги становятся кликабельными! Мы указали целевой ресурс 1 или нашу домашнюю страницу. Если ваш блог был на другой странице, кроме домашней, вы бы изменили ID там.

И, наконец, мы загружаем быстрый подсчет количества комментариев вместе с кликабельной ссылкой-меткой для их загрузки. Обратите внимание, что наше свойство 'thread' в вызове снипета  QuipCount (а затем и в вызове Quip) использует запись блога - [[*id]]. Это означает, что MODx будет автоматически создавать новую ветку для каждой новой записи блога, которую мы создаем.

### Содержание поста

Хорошо, вернемся к нашему шаблону. Сейчас мы находимся в разделе контента - обратите внимание, как мы начинаем с [[*introtext]]. Это полезное поле ресурса MODX - воспринимайте его как начальный отрывок для поста блога, который будет отображаться на нашей главной странице, когда мы публикуем последние посты блога.

### Добавление комментариев к посту

Хорошо, теперь мы находимся в части комментариев BlogPostTemplate. Как вы можете видеть здесь, мы используем [Quip](/extras/quip "Quip") для нашей системы комментирования. Вы можете использовать другую систему, такую как Disqus, здесь, если хотите. Для этого урока мы пойдем с Quip. Наш код выглядит следующим образом:

```php
<div class="post-comments" id="comments">[[!Quip?
   &thread=`blog-post-[[*id]]`
   &replyResourceId=`19`
   &closeAfter=`30`
 ]]
 <br /><br />
 [[!QuipReply?
    &thread=`blog-post-[[*id]]`
    &moderate=`1`
    &moderatorGroup=`Moderators`
    &closeAfter=`30`
 ]]
</div>
```

Хорошо. Обратите внимание, у нас есть два вызова снипетов здесь - один для отображения комментариев к этой теме ([Quip](/extras/quip/quip.quip "Quip.Quip")), и другой для отображения формы ответа ([QuipReply](/extras/quip/quip.quipreply "Quip.QuipReply")).

В нашем вызове снипета Quip мы указали идентификатор потока, как описано выше, а затем установили некоторые параметры. Наши комментарии будут иметь многопоточность (по умолчанию), поэтому нам нужно указать идентификатор ресурса, где будет находиться сообщение «Ответить в тему» (это подробно описано в[Quip документации](/extras/quip "Quip"). Мы рекомендуем прочитать там о том, как его настроить.) С 'replyResourceId'. Для примера, если ваш &replyResourceId указывает на страницу 123, то на странице 123 вы должны поместить что-то вроде следующего:

```php
[[!QuipReply]]
<br />
[[!Quip]]
```

Далее мы хотим указать - в вызовах Quip и Quip Reply - свойство 'closeAfter'. Это говорит Quip автоматически закрывать комментирование этих потоков после 30 дней создания потока (когда мы его загружаем).

В нашем вызове QuipReply мы хотим сказать Quip модерировать все сообщения, а модераторов для нашего сообщения можно найти в группе пользователей модераторов (мы объясним, как настроить это позже в руководстве).

Существует целый ряд других настроек Quip, которые мы можем изменить, но мы оставим вас для дальнейшей настройки, которую вы можете узнать, как это сделать в [Quip документации](/extras/quip "Quip").

**Что такое Threading?**
Если вы включите *threaded* комментарии, то пользователи могут комментировать другие комментарии. Непотоковые комментарии позволяют пользователям комментировать только исходное сообщение в блоге.

## Настройка тегов

Теперь, когда у нас есть все наши шаблоны, нам нужно настроить переменную шаблона 'tags', которую мы будем использовать для наших тегов.

Давайте создадим TV под названием 'tags' и дадим ему описание «теги с разделителями-запятыми для текущего ресурса» Затем убедитесь, что у него есть доступ к шаблону BlogPostTemplate, который мы создали ранее.

![](/download/attachments/18678105/tags-tv1.png?version=1&modificationDate=1279311829000)

Это оно! Теперь вы сможете добавлять теги к любому создаваемому нами сообщению в блоге, просто при редактировании своего ресурса, указав список тегов через запятую.

## Создание разделов

Если вы хотите, чтобы в вашем блоге были разделы (также называемые категориями), вам сначала нужно создать эти ресурсы.

Для этого урока мы создадим 2 раздела: «Личные» и «Технологии». Создайте 2 ресурса в корне вашего сайта и сделайте их «контейнерами». Вы захотите, чтобы их псевдоним был «личным» и «технологическим», поэтому URL-адреса вашего блога будут хорошо отображаться.

С этого момента мы скажем, что наши два Ресурса Секции имеют идентификаторы 34 и 35, для справки.

Убедитесь, что вы не используете BlogPostTemplate для них, а вместо этого используйте свой собственный базовый шаблон. Эти страницы станут способом просмотра всех сообщений в определенном разделе. В содержании этих Ресурсов добавьте следующее:

```php
[[!getResourcesTag?
 &element=`getResources`
 &elementClass=`modSnippet`
 &tpl=`blogPost`
 &hideContainers=`1`
 &pageVarKey=`page`
 &parents=`[[*id]]`
 &includeTVs=`1`
 &includeContent=`1`
]]
[[!+page.nav:notempty=`
<div class="paging">  
<ul class="pageList">  
 [[!+page.nav]]  
</ul>  
</div>
`]]
```

Хорошо, давайте объясним это: getResourcesTag фрагмент обёртки для [getResources](/extras/getresources "getResources") и [getPage](/extras/getpage "getPage") который автоматически фильтрует результаты по телевизору с тегами. Таким образом, в основном, мы хотим захватить все опубликованные ресурсы в этом разделе (и мы также можем фильтровать по тегам, если мы передадим в URL параметр '?tag=TagName').

Ниже вызова getResourcesTag мы размещаем наши ссылки на страницы, так как по умолчанию getResourcesTag показывает только 10 сообщений на страницу.

### Настройка чанка blogPost

В этом вызове у нас также есть свойство с именем 'tpl', которое мы установили в 'blogPost'. Это наш чанк, который показывает каждый результат наших записей в блогах. Он должен содержать это:

```php
<div class="post">
   <h2 class="title"><a href="[[~[[+id]]]]">[[+pagetitle]]</a></h2>
   <p class="post-info">Posted by [[+createdby:userinfo=`fullname`]]
[[+tv.tags:notempty=` | <span class="tags">Tags:
[[!tolinks? &items=`[[+tv.tags]]` &tagKey=`tags` &target=`1`]]
</span>`]]</p>
   <div class="entry">
       <p>[[+introtext]]</p>
   </div>
   <p class="postmeta">
     <span class="links">
<a href="[[~[[+id]]]]" class="readmore">Read more</a>
| <a href="[[~[[+id]]]]#comments" class="comments">
   Comments ([[!QuipCount? &thread=`blog-post-[[+id]]`]])
 </a>
| <span class="date">[[+publishedon:strtotime:date=`%b %d, %Y`]]</span>
     </span>
   </p>
</div>
```

Круто - давайте двигаться далее. Начнем с создания кликабельной ссылки на пост с заголовком страницы. Затем мы устанавливаем наш «опубликованный» элемент и список тегов (аналогично тому, как мы делали это ранее в BlogPostTemplate).

Далее мы покажем некоторые выдержки из содержимого, которые мы храним в поле 'introtext' содержимого.

После этого у нас есть симпатичная небольшая ссылка «читать дальше», которая ссылается на пост, а затем наши комментарии и дату публикации. Это оно!

![](/download/attachments/18678105/blogpost-tpl1.png?version=1&modificationDate=1279311929000)

## Настройка главной страницы вашего блога

На нашей домашней странице нашего блога, который мы получили в Resource ID 1 - начало нашего сайта - у нас есть это:

```php
[[!getResourcesTag?
 &elementClass=`modSnippet`
 &element=`getResources`
 &tpl=`blogPost`
 &parents=`34,35`
 &limit=`5`
 &includeContent=`1`
 &includeTVs=`1`
 &showHidden=`0`
 &hideContainers=`1`
 &cache=`0`
 &pageVarKey=`page`
]]
[[!+page.nav:notempty=`
<div class="paging">  
<ul class="pageList">  
 [[!+page.nav]]  
</ul>  
</div>
`]]
```

Это позволяет нам отображать все сообщения из двух разделов, которые мы сделали, в ресурсах 34 и 35. Это также позволяет нам фильтровать по тегам (поскольку все наши вызовы 'tolinks' и 'tagLister' имеют цель 1 (этот ресурс ID). Другими словами, поместив наш вызов getResourcesTag здесь, мы автоматически помечаем теги!

Вы можете легко сделать это другой страницей, чем ваш site_start (или ID 1) - просто убедитесь, что изменили свойства 'target' в вашем tagLister и использовали вызовы Snippet, чтобы отразить это.

## Добавление постов

Хорошо, теперь мы готовы фактически добавлять сообщения в блог, теперь, когда наша структура полностью настроена.

### Структура страницы в разделах

Прежде чем мы начнем, важно отметить, что то, как вы структурируете свои посты в этом разделе, полностью зависит от вас. Вы можете добавить год и месяц Контейнеры ресурсов, чтобы разместить эти посты, или просто разместить их непосредственно в разделе. Это полностью зависит от вас.

Если вы решите указать дату/год или субконтейнеры, убедитесь, что у них отмечен флажок Скрыть из меню, чтобы они не отображались в ваших вызовах getResources.

Помните, однако, что какую бы структуру вы ни построили по разделам, она не будет определять вашу навигацию -[Archivist](/extras/archivist "Archivist") справится с этим. Однако он определит URL ваших постов в блоге. Так что веселись.

### Добавление нового поста блога

Хорошо - продолжайте, создайте новый ресурс и установите для него шаблон «BlogPostTemplate». Тогда вы можете начать писать свой пост. Вы можете указать в поле 'introtext' выдержку для поста в блоге, а затем написать полное тело в поле контента.

И, наконец, когда вы закончите, убедитесь, что вы указали теги для своего поста в недавно созданном TV с тегами!

## Настройка ваших архивов

Отлично - у вас есть ваш первый пост в блоге! И у вас есть возможность просматривать его также в разделах. Теперь вам нужно настроить способ просмотра старых постов в блоге. Это где «Archvist» вступает в игру.

### Создание ресурса архивов

Идите вперед, поместите в свой корень ресурс под названием «Архивы» и дайте ему псевдоним 'archives'. Затем внутри содержимого разместите это:

```php
[[!getPage?
 &element=`getArchives`
 &elementClass=`modSnippet`
 &tpl=`blogPost`
 &hideContainers=`1`
 &pageVarKey=`page`
 &parents=`34,35`
 &includeTVs=`1`
 &toPlaceholder=`archives`
 &limit=`10`
 &cache=`0`
]]
<h3>[[+arc_month_name]] [[+arc_year]] Archives</h3>
[[+archives]]
[[!+page.nav:notempty=`
<div class="paging">  
<ul class="pageList">  
 [[!+page.nav]]  
</ul>  
</div>
`]]
```

Выглядит знакомо? Это очень похоже на getResourcesTag, описанный выше на нашей странице раздела. На этот раз getPage оборачивает снипет [getArchives](/extras/archivist "Archivist"), и говорит, что мы хотим получить посты в ресурсах 34 и 35 (страницы нашего раздела). Мы установим результат для заполнителя под названием «архивы», на который мы ссылаемся позже.

Затем, ниже этого, мы добавляем несколько плейсхолдеров, которые показывают текущий месяц и год просмотра. И наконец, у нас есть нумерация страниц. Здорово! Мы закончили с этим. Наш ресурс, для справочных целей, мы скажем, имеет идентификатор **30**.

### Настройка виджета Archivist

Итак, теперь у вас есть ресурс для просмотра архивов, но вам нужен какой-то способ генерирования месяцев, в которых перечислены сообщения. Это на самом деле довольно просто - где-то на вашем сайте (скажем, в вашем нижнем колонтитуле, поместите этот милый немного:

```php
<h3>Archives</h3>
<ul>
[[!Archivist? &target=`30` &parents=`34,35`]]
</ul>
```

Так что же снипет [Archivist](/extras/archivist/archivist.archivist "Archivist.Archivist") создает список сообщений за месяц (вы можете добавить другие варианты, но посмотрите [эту документацию](/extras/archivist/archivist.archivist "Archivist.Archivist") для этого). Мы говорим, что хотим, чтобы его ссылки шли на наш Архивный Ресурс (30), и собирали только посты в Ресурсах 34 и 35 (наши Ресурсы Раздела).

Это оно! Archivist фактически автоматически обработает все остальное - включая все ваши генерации URL для архивов - archives/2010/05/ покажет все посты в мае 2010 года, где archives /2009/ покажет все посты в 2009 году. Довольно мило, а?

## Расширенные настройки

### Добавление группы модераторов

Итак, ранее в нашем вызове QuipReply мы указали группу модераторов 'Moderators'. Давайте продолжим и создадим эту группу пользователей сейчас.

Перейдите в Безопасность -> Контроль доступа и создайте новую группу пользователей под названием «Модераторы». Добавьте в группу любых пользователей (включая себя!) И назначьте им любую роль.

Затем перейдите на вкладку «Доступ к контексту». Добавьте ACL (в основном строку), который предоставляет этой группе пользователей доступ в контексте «mgr» с минимальной ролью Member (9999), и политику доступа «QuipModeratorPolicy».

Это позволяет любому пользователю в группе «Модераторы» модерировать сообщения в ваших темах, а также уведомляет их по электронной почте о появлении новых сообщений. Затем они могут либо войти в систему менеджера, чтобы модерировать комментарии, либо щелкнуть ссылки прямо в письмах, чтобы одобрить или отклонить комментарии. Ваш ACL должен выглядеть примерно так:

![](/download/attachments/18678105/moderator-group.png?version=1&modificationDate=1279308769000)

Сохраните свою группу пользователей, и все! Возможно, вам придется сбросить сессии (Безопасность -> Сброс сессий) и повторно войти в систему, чтобы перезагрузить ваши разрешения, но Quip будет обрабатывать остальное.

### Добавление виджета "Последние сообщения"

Возможно, вам понадобятся «Последние сообщения» где-то на сайте, и не бойтесь - добавить их довольно просто.

Во-первых, вы захотите сделать этот звонок везде, где вы хотите, чтобы появился список:

```php
[[!getResources?
 &parents=`34,35`
 &hideContainers=`1`
 &tpl=`latestPostsTpl`
 &limit=`5`
 &sortby=`publishedon`
]]
```

Итак, мы говорим [getResources](/extras/getresources "getResources") отобразить список из 5 лучших ресурсов в разделе ресурсов (34,35) и отсортировать их по дате публикации.

Затем создайте чанк `latestPostsTpl`, который вы указали с помощью вызова 'tpl' в вызове сниппета getResources. Поместите это как содержимое чанка:

```php
<li>
 <a href="[[~[[+id]]]]">[[+pagetitle]]</a>
 [[+publishedon:notempty=`<br /> - [[+publishedon:strtotime:date=`%b %d, %Y`]]`]]
</li>
```

И бум! Последние записи блога, отображаемые на вашем сайте:

![](/download/attachments/18678105/latestposts.png?version=1&modificationDate=1279308815000)

### Добавление виджета «Последние комментарии»

А как насчет виджета, который показывает несколько последних комментариев в ваших сообщениях? Просто - Quip вызываает маленький снипет под названием [QuipLatestComments](/extras/quip/quip.quiplatestcomments "Quip.QuipLatestComments") который с этим легко справиться.

Разместите вызов там, где вы хотите, чтобы список комментариев отображался:

```php
[[!QuipLatestComments? &tpl=`latestCommentTpl`]]
```

Теперь создайте чанк с именем 'latestCommentTpl':

```php
<li class="[[+cls]][[+alt]]">
   <a href="[[+url]]">[[+body:ellipsis=`[[+bodyLimit]]`]]</a>
   <br /><span class="author">by [[+name]]</span>
   <br /><span class="ago">[[+createdon:ago]]</span>
</li>
```

Прежде чем мы продолжим, следует отметить несколько вещей: QuipLatestComments автоматически обрежет комментарий и добавит многоточие после переданного в него свойства &bodyLimit, значение которого по умолчанию равно 30 символам. Во-вторых, обратите внимание на использованный здесь «назад» [фильтр вывода ](building-sites/tag-syntax/output-filters) «Фильтры ввода и вывода (модификаторы вывода)»). Этот фильтр встроен в MODX Revolution и переводит временную метку в красивый, симпатичный формат «два часа, 34 минуты» (или две другие метрики времени, такие как мин/сек, год/месяц, месяц/месяц).

Обратите внимание, что по умолчанию будет отображаться 5 последних. Результат:

![](/download/attachments/18678105/latestcomments.png?version=1&modificationDate=1279309243000)

Вы можете посмотреть [документацию для снипета](/extras/quip/quip.quiplatestcomments "Quip.QuipLatestComments") для получения дополнительных параметров конфигурации.

### Добавление виджета «Самые популярные теги»

Эта часть смехотворно проста [tagLister](/extras/taglister "tagLister") делает это для вас. Просто поместите это куда хотите

```php
[[!tagLister? &tv=`tags` &target=`1`]]
```

А tagLister проверит TV 'tags' и создаст ссылки, которые идут к цели (здесь ID ресурса 1) с использованием 10 лучших тегов. Есть еще [варианты конфигурации](/extras/taglister "tagLister"), но мы оставим вас с этим.

## Заключение

Итак, у нас есть полная настройка блога! Теперь это должно выглядеть примерно так в нашем дереве:

![](/download/attachments/18678105/blog-tree2.png?version=1&modificationDate=1279307462000)

Опять же, есть гораздо больше настроек и вещей, которые вы можете добавить в свой блог. Это учебное пособие задумано как отправная точка, но вы можете свободно настраивать и добавлять вещи по своему вкусу - главное в MODX заключается в том, что вы можете очень легко настраивать, настраивать и масштабировать любое решение, включая блог!

Помните, этот урок был основан на [splittingred.com](http://splittingred.com), если вы хотите увидеть полномасштабную демонстрацию этого в действии.