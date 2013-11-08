# PSG-protocol (Platform shooter game protocol)

Протокол обмена сообщениями между клиентом и сервером

Сообщения передаются в формате JSON методом POST

## Содержание

* [PSG-protocol (Platform shooter game protocol)](#psg-protocol-platform-shooter-game-protocol)
    * [Содержание](#Содержание)
    * [Формат сообщений](#Формат-сообщений)
        * [Запрос (Request) ](#Запрос-request)
            * [Пример](#Пример)
        * [Ответ (Response) ](#Ответ-response)
            * [Базовые коды ошибок, расположенные по убыванию приоритета](#Базовые-коды-ошибок-расположенные-по-убыванию-приоритета)
                * [Примечание](#Примечание)
            * [Пример ответа в случае успеха](#Пример-ответа-в-случае-успеха)
            * [Пример ответа в случае ошибки](#Пример-ответа-в-случае-ошибки)
    * [Действия](#Действия)
        * [startTesting](#starttesting)
        * [signup](#signup)
        * [signin](#signin)
        * [signout](#signout)
        * [sendMessage](#sendmessage)
        * [getMessages](#getmessages)
            * [Пример массива messages](#Пример-массива-messages)
        * [createGame](#creategame)
            * [Примечание](#Примечание-1)
        * [getGames](#getgames)
            * [Пример массива games](#Пример-массива-games)
        * [joinGame](#joingame)
        * [leaveGame](#leavegame)
            * [Примечание](#Примечание-2)
        * [uploadMap](#uploadmap)
            * [Примечание](#Примечание-3)
            * [Спецификация карты](#Спецификация-карты)
        * [getMaps](#getmaps)
            * [Пример массива maps](#Пример-массива-maps)
    * [Websocket](#websocket)
        * [Игровые клетки](#Игровые-клетки)
        * [Система координат](#Система-координат)
        * [Ограничения](#Ограничения)
        * [Физика](#Физика)
        * [Использование респаунов и телепортов](#Использование-респаунов-и-телепортов)
            * [Пример использования респаунов](#Пример-использования-респаунов)
        * [Запросы, требующие websocket](#Запросы-требующие-websocket)
            * [Сообщение, которое сервер рассылает по всем соединениям websocket каждый тик](#Сообщение-которое-сервер-рассылает-по-всем-соединениям-websocket-каждый-тик)
            * [Общие параметры сообщений, которые отправляет клиент](#Общие-параметры-сообщений-которые-отправляет-клиент)
            * [move](#move)
            * [fire](#fire)
            * [Пример сообщения клиента](#Пример-сообщения-клиента)

## Формат сообщений

### Запрос (Request)

* action => строка, название действия
* params => объект, описывающий параметры, специфичные для этого действия. *Необязательный в том случае, когда действие не подразумевает ни одного параметра (Например, для действия startTesting)*

#### Пример

    {
        "action": "signup",
        "params": {
            "login": "admin",
            "password": "banana"
        }
    }


### Ответ (Response)

* result => строка, результат выполнения действия
    - В случае успешного выполнения действия значение поля result должно быть равно "ok"
    - В случае ошибки передается ее код. Если ошибок несколько, то передается наиболее приоритетная из них. Ошибки с одинаковым приоритетом выбираются произвольно

* message => *строка, необязательный текстовый комментарий. Может быть произвольным*

#### Базовые коды ошибок, расположенные по убыванию приоритета

* badJSON (Запрос не соответствует [стандарту JSON](http://www.json.org/json-ru.html))
* badRequest (Какое-либо обязательное поле не было обнаружено, т.е. нарушен формат сообщений)
* badAction (Команда не найдена либо поле action некорректно)

##### Примечание

* Ошибки типа bad&lt;Property&gt;, т.е. описывающие несоответствие параметров ограничениям, имеют больший приоритет среди специфичных для запросов. *Например, в действии signin более приоритетной будет ошибка "badLogin" над "incorrect"*
* В общем случае, приоритеты ошибок расставлены следующим образом (т.е. ошибки, относящиеся к одному пункту из следующего списка. имеют одинаковый приоритет), по убыванию:
    - badJSON
    - badRequest
    - badAction
    - bad&lt;Property&gt;
    - все остальные
* Если в запросе пришло больше параметров, чем требуется, лишние параметры следует игнорировать

#### Пример ответа в случае успеха

    {
        "result": "ok"
    }

#### Пример ответа в случае ошибки

    {
        "result": "userExists",
        "message": "Такой пользователь уже существует"
    }


## Действия

### startTesting

Коды ошибок:

- notInTestMode (Сервер не в тестирующем режиме)

### signup

Параметры:

* login => строковое значение.
    - минимальная длина: 4 символа
    - макисмальная длина: 40 символов
* password => строковое значение.
    - минимальная длина: 4 символа

Коды ошибок:

* userExists (Такой пользователь уже существует)
* badLogin (Некорректный логин)
* badPassword (Некорректный пароль)

### signin

Параметры:

* login => строковое значение
* password => строковое значение

Ответ:

* sid => латинские буквы и цифры, не короче 1 символа

Коды ошибок:

* incorrect (Не найден пользователь с таким логином и паролем)
* badLogin (Логин не является строкой)
* badPassword (Пароль не является строкой)

### signout

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)

---

### sendMessage

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа
* game => число, идентификатор игры. Пустая строка -- для общего чата
* text => текст сообщения, строковое значение

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)
* badGame (Некорректный идентификатор игры либо пользователь не может отправлять сообщение в эту игру)
* badText (Некорректный текст сообщения)

### getMessages

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа
* game => число, идентификатор игры. Пустая строка -- для общего чата
* since => Unix timestamp UTC

Ответ:

* messages => массив объектов, описывающих сообщения, отсортированный *по возрастанию* времени получения сообщения на сервере
    - time => время получения сообщения на сервере в формате Unix timestamp UTC. Должно быть не меньше параметра since
    - text => строка, текст сообщения
    - login => строка, логин отправителя сообщения

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)
* badGame (Некорректный идентификатор игры либо пользователь не может получать сообщения из этой игры)
* badSince (Некорректное время)

#### Пример массива messages

    [
        {
            "time": 1382454174,
            "text": "Hello there",
            "login": "OldGuy1915"
        },
        {
            "time": 1382713375,
            "text": "Are you still here?",
            "login": "Newbie"
        }
    ]

---

### createGame

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа
* name => название игры, строковое значение, не короче одного символа
* map => число, идентификатор карты
* maxPlayers => максимально допустимое число игроков

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)
* badName (Некорректное название игры)
* badMap (Некорректный идентификатор карты либо карта с таким идентификатором не найдена)
* badMaxPlayers (Некорректное максимальное число игроков либо оно меньше допустимого картой)
* gameExists (Игра с таким названием уже существует)
* alreadyInGame (Пользователь, находящийся в какой-либо игре, не может создавать игры)

#### Примечание

* Пользователь, создавший игру, автоматически в нее заходит (т.е. не требуется отправка дополнительного запроса на joinGame)
* При попытке два раза подряд отправить запрос createGame с одними и теми же аргументами, возникает две ошибки (alreadyInGame и gameExists). Соответственно, корректным будет считаться ответ, в котором содержится любая из них
* Допускается создание игр, использующих одну и ту же карту
* Максимальное количество игроков, допустимое для игры, не должно превышать максимальное количество игроков, предусмотренных картой

### getGames

Параметры:

* sid => латинские буквы и цифры, не короче одного символа

Ответ:

* games => массив объектов, описывающих игры. Порядок произвольный
    - id => число, идентификатор игры
    - name => строка, название игры
    - map => число, идентификатор карты
    - maxPlayers => число, максимальное количество игроков для этой игры
    - players => массив логинов пользователей, участвующих в этой игре, отсортированный в порядке их подключения к игре
    - status => строковое значение, состояние игры
        + Допустимые значения: "running", "finished"

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)

#### Пример массива games

    [
        {
            "id": 0,
            "name": "Super cool game",
            "map": 1,
            "maxPlayers": 12,
            "players": [
                "Creator", "User1"
            ],
            "status": "running"
        },
        {
            "id": 2,
            "name": "Platformer cup 2013",
            "map": 12,
            "maxPlayers": 4,
            "players": [
                "android lover", "**apple-fan-boy**", "windows_user313"
            ],
            "status": "finished"
        },
        {
            id: 123,
            "name": "Dm",
            "map": 1,
            "maxPlayers": 5,
            "players": [
                "Lonely guy"
            ],
            "status": "running"
        }
    ]

### joinGame

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа
* game => число, идентификатор игры

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)
* gameFull (Игра заполнена)
* badGame (Некорректный идентификатор игры)
* alreadyInGame (Пользователь, находящийся в какой-либо игре, не может подключаться к играм (в том числе к своей))

### leaveGame

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа

Коды ошибок:

* notInGame (Игрок не участвует ни в одной игре)

#### Примечание

После выхода последнего игрока, игра должна быть удалена

### uploadMap

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа
* name => название игры, строковое значение
* map: ["<row1>", "<row2>", ...] => массив строк, описывающих карту. Длины строк должны быть равны
* maxPlayers => максимальное число игроков для этой карты, число

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)
* badName (Некорректное название карты)
* badMap (Некорректные данные карты)
* badMaxPlayers (Некорректное максимальное число игроков)
* mapExists (Карта с таким названием уже существует)

#### Примечание

* Допускается создание карт, использующих разные названия, но одинаковые массивы строк map

#### Спецификация карты

* Элементы массива, описывающего карту, должны быть одинаковой длины (Карта прямоугольная)
* Может быть только два телепорта, помеченных одной цифрой. Они считаются соответственно входом и выходом.
* Одной клетке соответствует 1 символ

Допустимые символы:

+ ```.``` - пустое место
+ ```$``` - место респауна
+ ```#``` - стенка (считается, что стенки непробиваемые)
+ [a-z] - предметы. *Пока про предметы не договорились*
+ [A-Z] - оружиe:
    - ```K``` - knife
    - ```P``` - pistol
    - ```M``` - machine gun
    - ```R``` - rocket launcher
    - ```A``` - railgun
+ [0-9] - телепорты (Одинаковые цифры обозначают входы и выходы одного телепорта)

Пример:

    .2.....1....2.
    .#R..$.1.M..#.
    .############.
    ..............

Минимальный размер карты - 1 x 1 клетка

### getMaps

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа

Ответ:

* maps => массив объектов, описывающих карты. Порядок произвольный
    - id => число, идентификатор карты
    - name => строка, название карты
    - maxPlayers => число, максимально допустимое количество игроков для этой карты
    - map => данные карты в виде массива строк

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)

#### Пример массива maps

    [
        {
            "id": 1,
            "name": "Small map",
            "maxPlayers": 2,
            "map": [
                ".$",
                "##"
            ]
        },
        {
            "id": 500,
            "name": "Another map",
            "maxPlayers": 3,
            "map": [
                "#$.......$#",
                "###########"
            ]
        }
    ]


## Websocket

### Игровые клетки

* Клетка -- квадрат со стороной 1
* Игрок -- квадрат размером с клетку
* Координаты игрока -- координаты центра квадрата игрока

### Система координат

* координаты 0, 0 соответствуют левой верхней вершине левой верхней клетки
* x направлен вправо
* y направлен вниз

Допустимая точность координат -- 6 знаков после запятой

### Ограничения

* Игровой тик - 30 мс
* Максимальная скорость передвижения по каждой компоненте -- 1 клетка за тик
* Изменение скорости по модулю при движении -- 0.1 за тик

### Физика

* Прыжок можно осуществлять только, когда игрок стоит на опоре (стенке)
* При прыжках и падении к вертикальной составляющей скорости за каждый тик должна прибавляться константа 0.2, что характеризует гравитацию
* Столкновение с любой стенкой/границей карты является абсолютно неупругим, т.е. при столкновении с вертикальной стенкой обнуляется скорость по x, при столкновении с горизонтальной - по y
* Считает, что игрок столкнулся со стенкой/границей карты, если единичный квадрат игрока пересекается с квадратом стены/выходит за пределы карты.
* При прыжке (т.е. когда от клиента отправляется ненулевой dy и это допустимый случай для прыжка), скорость по y следует делать максимально допустимой *по модулю* -- скорость при прыжке изначально будет отрицательной по y
* Если от клиента за тик приходит несколько команд move, то следует сначала посчитать результирующую скорость, а затем вывести из этих данных следующую координату
* Если игрок находится на опоре и от клиента не приходит за тик ни одного мува, то следует начать его торможение, то есть изменять скорость в противоположном направлении его движению согласно [ограничениям](#Ограничения) либо делать равной нулю, если изменение скорости больше по модулю, чем сама скорость игрока.

### Использование респаунов и телепортов

Порядок обхода респаунов и телепортов одной сети -- слева направа, сверху вниз по кругу

* Если персонаж появился из какого-то респауна, считается, что этот респаун - последний использованный. Каждое перерождение происходит в следующем за последним использованным респауном.
    - Из респауна игрок появляется без движения
* Происходит телепортация игрока, если во время движения единичный квадрат игрока пересекся с координатами центра телепорта.
* При телепортации игрока его скорость остается неизменной, а координаты изменяются на координаты центра выхода из телепорта.

#### Пример использования респаунов

Рассмотрим карту:

    ..$......$.
    .###...###.
    ...$.......
    .######....

Порядок появлений из респаунов будет следующий:

1. Третья клетка первой строки
2. Предпоследняя клетка первой строки
3. Четвертая клетка третьей строки
4. Третья клетка первой строки
5. И так далее

---

### Запросы, требующие websocket

Сообщения, требующие websocket, предлагается отправлять на /websocket (Например, http://localhost/websocket, если запросы без websocket отправлялись на localhost)

При установлении websocket-соединения, первым запросом требуется отправить move с нулевыми параметрами dx, dy

#### Сообщение, которое сервер рассылает по всем соединениям websocket каждый тик

* tick => число, номер игрового тика
* players => массив объектов, описывающих каждого игрока. В порядке подключения к игре
    - x => число, координата x игрока
    - y => число, координата y игрока
    - vx => число, скорость по оси x игрока
    - vy => число, скорость по оси y игрока
    - health => число, процент здоровья игрока
    - status => "dead" или "alive" - статус игрока
    - respawn => число, время в миллисекундах до респауна игрока. В случае, если status == "alive", поле не требуется
* projectiles => массив объектов, описывающих снаряды
    - x => число, x-компонента координат снаряда
    - y => число, y-компонента координат снаряда
    - vx => число, скорость по оси x снаряда
    - vy => число, скорость по оси y снаряда
    - owner => число, индекс игрока в массиве players, который сделал выстрел
* items => массив, каждый элемент - время до восстановления i-го предмета на карте. Предметы нумеруются относительно расположения на карте слева направо сверху вниз

#### Общие параметры сообщений, которые отправляет клиент

* sid => латинские буквы и цифры, не менее 1 символа
* tick => число, номер игрового тика

#### move

Параметры:

* dx => число, смещение игрока по x
* dy => число, смещение игрока по y

#### fire

Параметры:

* dx => число, смещение координат выстрела по x
* dy => число, смещение координат выстрела по y

#### Пример сообщения клиента

    {
        "params": {
            "sid": "usersid",
            "tick": 10,
            "dx": 12.123,
            "dy": 123.123
        },

        "action": "move"
    }
