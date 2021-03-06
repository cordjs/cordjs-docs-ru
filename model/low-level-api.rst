****************************************************************
Низкоуровневые ReST API запросы --- работа с данными без моделей
****************************************************************

CordJS предоставляет готовый сервис для осуществления ReST API запросов к бекенду. Этот сервис можно внедрить как
зависимость к любому виджету по имени ``api``. Он поддерживает аутентификацию по стандарту OAuth2.

В большинстве случае следует избегать низкоуровневых запросов к бекенду и использовать подсистему моделей, но иногда
это необходимо.


Кросс-доменный прокси
=====================

Поскольку бекенд-сервис, зачастую, может использовать домен, отличный от домена фроненд-приложения, то необходимо
уметь обходить ограничения браузера по кросс-доменным запросам. CordJS использует для этого кросс-доменный
прокси-сервер, реализованный в серверной части, работающей на Node.js. Запросы из браузера идут сначала к серверу
фронтенд-приложения и перенаправляются уже на бекенд-сервис. Во время рендеринга страницы на сервере такой проблемы
нет, и запросы на бекенд идут напрямую.


Сервис ``api``
==============

Настройки
---------

Сервис ``api`` поддерживает несколько настроек, позволяющих упростить работу:

* ``api.protocol`` --- строка ``http`` или ``https``. Протокол, который следует использовать для запроса.
* ``api.host`` --- название хоста бекенд-сервера (можно с портом), к которому нужно делать запрос.
* ``api.urlPrefix`` --- префикс URL (без начального слеша), который добавляется после хоста в каждый API-запрос.
  Например, ``api/v1/``.

Для серверной и браузерной части эти настройки разные, поскольку браузер в большинстве случаев должен пользоватся
кросс-доменным прокси-сервером.


API
---

``get``, ``post``, ``put``, ``del`` --- выполнить запрос к бекенд серверу
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

У сервиса ``api`` есть единый метод ``send``, который выполняет запрос. Однако обычно им не пользуются, а пользуются
алиасами ``get``, ``post``, ``put`` или ``del``, которые выполняют запрос сразу с нужным http-методом (глаголом). Эти
методы принимают следующие аргументы:

* *Строка* ``url`` --- кусок URL, добавляемый после ``api.protocol``, ``api.host`` и ``api.urlPrefix``. Обычно это
  путь к ReST-ресурсу. По такому собранному из настроек и этого аргумента URL'у делается запрос.

* *Объект* ``params`` --- параметры запроса в форме ключ-значение. Среди них могут быть специальные параметры,
  управляющие логикой выполнения зарпроса:

  * *Логическое* ``noAuthTockens`` --- если значение ``true``, то не добавлять авторизационные токены к параметрам
    запроса. По умолчанию они добавляются в соответствие с настройками аутентификации.

  * *Логическое* ``skipAuth`` --- если значение ``true``, то не пытаться авторизовать пользователя (например,
    редиректить на форму логина) в случае ошибки запроса из-за ошибки авторизации.

  * *Целое* ``retryCount`` --- максимальное количество попыток выполнить запрос с паузами в 200 миллисекунд в случае
    ошибки. Значение по умолчанию --- 3.


Поддержка аутентификации через OAuth2
=====================================

API-запросы в CordJS по умолчанию требуют аутентификации. Модули аутентификации могут быть разными, но основная
реализация в CordJS --- аутентификация по протоколу OAuth2. Бекенд должен реализовать поддержку такой аутентификации,
а URL, с помощью которого происходит получение токена авторизации, задаётся настройкой
``api.oauth2.endpoints.accessToken``.


``authByUsernamePassword`` --- метод для входа по введённым логину и паролю
---------------------------------------------------------------------------

Метод сервиса ``api``, который используется на страницах с формой логина для инициализации авторизационной сессии по
логину и паролю. Принимает на вход 2 соответствующих строковых поля (в чистом виде, без шифровки). Возвращает пустой
:term:`промис`, который резолвится, когда авторизация успешно прошла и токены получены.


``authTokensAvailable`` --- проверка наличия авторизационных токенов
--------------------------------------------------------------------

Этот метод сервиса ``api`` проверить принципиальное наличие сохранённых токенов аутентификации. Он не принимает на
вход аргументов и возвращает ``true`` или ``false``, завёрнутый в промис. Результат ``true`` не гарантирует, что
сохранённые токены аутентификации не устарели и позволят выполнить запрос успешно.
