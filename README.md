Implementation of some DPI bypass methods.
The program is a local SOCKS proxy server.

Usage example:
```
ciadpi --disorder 1 --auto=torst --tlsrec 1+s
ciadpi --fake -1 --ttl 8
```

------
### Описание аргументов
```
-i, --ip <ip>
    Прослушиваемый IP, по умолчанию 0.0.0.0

-p, --port <num>
    Прослушиваемый порт, по умолчанию 1080

-D, --daemon
    Запуск в режиме демона
    Поддерживается только в Linux и BSD системах

-w, --pidfile <filename>
    Расположение PID-файла

-E, --transparent
    Запуск в режиме прозрачного прокси, SOCKS работать не будет
    
-c, --max-conn <count>
    Максимальное количество клиентских подключений, по умолчанию 512

-I,  --conn-ip <ip>
    Адрес, к которому будут привязаны исходящие соединения, по умолчанию ::
    При указании IPv4 адреса запросы на IPv6 будут отклоняться

-b, --buf-size <size>
    Максимальный размер данных, получаемых и отправляемых за один вызов recv/send
    Размер указывается в байтах, по умолчанию равен 16384

-g, --def-ttl <num>
    Значение TTL для всех исходящий соединений
    Может быть полезен для обхода обнаружения нестандартного/уменьшенного TTL

-N, --no-domain
    Отбрасывать запросы, если в качестве адреса указан домен
    Т.к. резолвинг выполняется синхронно, то он может замедлить или даже заморозить работу

-U, --no-udp
    Не проксировать UDP
    
-F, --tfo
    Включает TCP Fast Open
    Если сервер его поддерживает, то первый пакет будет отправлен сразу вместе с SYN
    Поддерживается только в Linux (4.11+)
    
-A, --auto <t,r,s,n>
    Автоматический режим
    Если произошло событие, похожее на блокировку или поломку,
    то будут применены параметры обхода, следующие за данной опцией
    Возможные события:
        torst   : Вышло время ожидания или сервер сбросил подключение после первого запроса
        redirect: HTTP Redirect с Location, домен которого не совпадает с исходящим
        ssl_err : В ответ на ClientHello не пришел ServerHello или SH содержит некорректный session_id
        none    : Предыдущая группа пропущена, например из-за ограничения по доменам или протоколам
    
-L, --auto-mode <0|1>
    0: кешировать IP только если имеется возможность переподключиться
    1: кешировать IP также в том случае, если:
        torst - таймаут/соединение сброшено во время обмена пакетами (т.е. уже после первых данных от сервера)
        ssl_err - совершился лишь один круг обмена данными (запрос-ответ/запрос-ответ-запрос)
    
-u, --cache-ttl <sec>
    Время жизни значения в кеше, по умолчанию 100800 (28 часов)
    
-T, --timeout <sec>
    Таймаут ожидания первого ответа от сервера в секундах
    В Linux переводится в миллисекунды, поэтому можно указать дробное число
    
-K, --proto <t,h,u,i>
    Белый список протоколов: tls,http,udp,ipv4
    
-H, --hosts <file|:string>
    Ограничить область действия параметров списком доменов
    Домены должны быть разделены новой строкой или пробелом
    
-j, --ipset <file|:str>
    Ограничитель по определенным IP/подсетям
    
-V, --pf <port[-portr]>
    Ограничитель по портам
    
-R, --round <num[-numr]>
    К каким/какому запросу применять запутывание
    По умолчанию 1, т.е. к первому запросу
    
-s, --split <pos_t>
    Разбить запрос по указанной позиции
    Позиция имеет вид offset[:repeats:skip][+flag1[flag2]]
    Флаги:
        +s: добавить смещение SNI
        +h: добавить смещение Host
        +n: нулевое смещение
    Дополнительные флаги:
        +e: конец; +m: середина
    Примеры: 
        0+sm - разбить запрос в середине SNI
        1:3:5 - разбить по позициям 1, 6 и 11
    Ключ можно указывать несколько раз, чтобы разбить запрос по нескольким позициям
    Если offset отрицательный и не имеет флагов, то к нему прибавляется размер пакета
    
-d, --disorder <pos_t>
    Подобен --split, но части отправляются в обратном порядке
    
-o, --oob <pos_t>
    Подобен --split, но часть отсылается как OOB данные
    
-q, --disoob <pos_t>
    Подобен --disorder, но часть отсылается как OOB данные
    
-f, --fake <pos_t>
    Подобен --disorder, только перед отправкой первого куска отправляется часть поддельного
    Количество байт отправляемого из фейка равно рамеру разбиваемой части
    ! На Windows может работать нестабильно, может заморозить работу прокси на некоторое время, если фейки шлются одновременно на нескольких соединениях
 
-t, --ttl <num>
    TTL для поддельного пакета, по умолчанию 8
    Необходимо подобрать такое значение, чтобы пакет не дошел до сервера, но был обработан DPI

-k, --ip-opt[=file|:str]
    Установить опции для фейкового IP пакета
    Существенно снизит вероятность, что пакет дойдет до сервера
    Стоит учесть, что до DPI он также может не дойти
    В Windows не поддерживается
    
-S, --md5sig
    Установить опцию TCP MD5 Signature для фейкового пакета
    Большинство серверов (в основном на Linux) отбрасывают пакеты с данной опцией
    Поддерживается только в Linux, может быть выключен в некоторых сборках ядра (< 3.9, Android)

-O, --fake-offset <n>
    Сместить начало фейковых данных на n байт
       
-l, --fake-data <file|:str>
    Указать свои поддельные пакеты
    Строка может содержать escape символы (\n,\0,\0x10)

-e, --oob-data <char>
    Байт, отсылаемый вне основного потока, по умолчанию 'a'
    Можно указать ASCII или escape символ
    
-n, --tls-sni <str>
    Изменить SNI в дефолтном fake пакете на указанный

-M, --mod-http <h[,d,r]>
    Всякие манипуляции с HTTP пакетом, можно комбинировать
    hcsmix:
        "Host: name" -> "hOsT: name"
    dcsmix:
        "Host: name" -> "Host: NaMe"
    rmspace:
        "Host: name" -> "Host:name\t"

-r, --tlsrec <pos_t>
    Разделить ClientHello на отдельные записи по указанному смещению
    Можно указывать несколько раз  

-a, --udp-fake <count>
    Количество фейковых UDP пакетов

-Y, --drop-sack
    Игнорировать SACK, вынуждая ядро переотправить уже доставленные пакеты
    Поддерживается только в Linux
```

------
### Подробнее
`--split`

Разбивает запрос на части. Пример на запросе в 30 байт:
- Параметры: `--split 3 --split 7`
- Порядок отправки: 1-3, 3-7, 7-30  

Позиции следует указывать в порядке возрастания.  

------
`--disorder`

Часть, попадающая под disorder, будет отправлена с TTL=1, т.е. фактически не будет никуда доставлена.
ОС узнает об этом лишь после отсылки последующей части, когда сервер сообщит о потере с помощью SACK.
Системе придется отослать предыдущий пакет заново, тем самым нарушив обычный порядок.
- Параметры: `--disorder 7`
- Порядок отправки: 7-30, 1-7  

Вышесказанное распространяется только на Linux.
В Windows ретрансмиссия начинается с позиции, с которой начались потери (максимальный ACK, полученный от сервера):
- Параметры: `--disorder 7`
- Порядок отправки: 7-30, 1-30

Поэтому желательно использовать ещё и `split`:  
- Параметры: `--split 7 --disorder 23`
- Порядок отправки: 1-7, 23-30, 7-30

На практике оптимально использовать:  
* Linux: `--disorder 1`
* Windows: `--split 1+s --disorder 3+s`

------
`--fake`

- Параметры: `--fake 7`
- Порядок отправки: 1-7 фейк, 7-30 оригинал, 1-7 оригинал

Данные в первой части запроса заменяются на поддельные.  
Эта часть должна пройти через DPI, но не дойти до сервера.
А раз часть не дойдет, то ОС отправит ее снова, тем самым изменив порядок подобно `disorder`.
Для того, чтобы фейк не дошел до сервера, есть опции `ttl`, `ip-opt` и `md5sig`.  

TTL необходимо подбирать такой, чтобы пакет прошел через все DPI, но не дошел до сервера.  
Для Linux есть md5sig. Он устанавливает опцию TCP MD5 Signature, что не дает пакету быть принятым многими серверами.
К сожалению, md5sig работает не во всех сборках.  

Для Windows есть еще один способ избежать обработки фейка сервером.
Это комбинирование `fake` с `disorder`:
- Параметры: `--disorder 1 --fake 7`
- Порядок отправки: 2-7 фейк, 7-30 оригинал, 1-30 оригинал  

Если поддельный пакет и дойдет до сервера, то он будет перезаписан из-за полной ретрансмисси.  

На практике оптимально использовать:  
* Linux: `--fake -1 --md5sig`
* Windows: `--disorder 1 --fake -1`

------
`--oob`

TCP может отсылать данные вне основного потока, используя флаг URG, однако лишь 1 байт в пакете.  
Все данные в таком пакете будут доставлены приложению, кроме последнего байта, который и является внеканальным:
- Параметры: `--oob 3`
- Отправка: 1-4 с флагом URG (1-3 данные запроса + 4-й байт, который будет усечен), 3-30

Этот байт желательно помещать в SNI: `--oob 3+s` 

------
`--disoob`

Схож с `--disorder`, но часть отправляется с OOB байтом:
- Параметры: `--disoob 3`
- Отправка: 3-30, 1-4 с флагом URG (1-3 данные запроса + 4-й байт, который будет усечен)

При использовании с `--fake` или `--disorder` можно получить пакет, где OOB байт будет находиться на месте разбиения:
- Параметры: `--disoob 3 --disorder 7`
- Отправка: 3-30, 1-8 с флагом URG (1-3 + байт который будет усечен + 4-8)

------
`--tlsrec`

Одну TLS запись можно разбить на несколько, немного переделав заголовок.  
На месте разбиения вставляется новый заголовок, увеличивая размер запроса на 5 байт.  

Этот заголовок можно поместить в середину SNI, не давая возможность DPI правильно его прочитать: 
`--tlsrec 3+s`

Хоть `tlsrec` и `oob` запутывают DPI, они также могут запутать всякие мидлбоксы, которые не поддерживают полноценный стек TCP/TLS.  
Из-за этого их следует использовать вместе с `--auto`:  
`--auto=torst --timeout 3 --tlsrec 3+s`  
В примере `tlsrec` будет применяться лишь в случаях, когда сброшено подключение или вышел таймаут, т.е. когда, скорее всего, произошла блокировка.  
Можно наоборот - отменять tlsrec, если сервер сбрасывает подключение или откидывает пакет:  
`--tlsrec 3+s --auto=torst --timeout 3`  

------
`-Y, --drop-sack`

Заставляет ядро игнорировать пакеты с расширением TCP SACK.
Это расширение позволяет подтверждать получение отдельных сегментов данных.
Если первая часть запроса будет потеряна, а до сервера дойдет лишь вторая, то сервер с помощью этого расширения может уведомить клиента об этом. Тогда клиент, зная, что вторая часть дошла, отправит лишь первую.  
Зачем игнорировать это расширение? Второй сегмент может быть фейковым. Если он дойдет до сервера, но клиент об этом не узнает, то он попытается переотправить его. Однако этот сегмент будет содержать уже оригинальные данные, которые перезапишут фейковые, тем самым предотвратив поломку протокола.  
Так как быстрое подтверждение работать не будет, то это сломает `disorder`, а также добавит задержку перед ретрансмиссией (около 200ms).

------
`--auto`, `--hosts`

Параметр `auto` делит опции на группы.
Для каждого запроса они обходятся слева на право.
Сначала проверяется триггер, указанный в `auto`, затем `pf`, `proto` и `hosts`.

Можно указывать несколько групп опций, раделяя их данным параметром.  
Параметры, которые идут ниже `--timeout` в help-тексте (кроме `tls-sni`), можно вынести в отдельную группу.  

#### Примеры:
```
--fake -1 --ttl 10 --auto=ssl_err --fake -1 --ttl 5
```
По умолчанию использовать `fake` с ttl=10, в случае ошибки использовать `fake` с ttl=5

```
--hosts list.txt --disorder 3 --auto=none
```
Применять запутывание только для доменов из list.txt

```
--hosts list.txt --auto=none --disorder 3
```
Не применять запутывание для доменов из list.txt

```
--auto=torst --hosts list.txt --disorder 3
```
По умолчанию ничего не делать, использовать disorder при условии, что произошла блокировка и домен входит в list.txt.

```
--proto=http,tls --disorder 3 --auto=none
```
Запутывать только HTTP и TLS

```
--proto=http --fake -1 --fake-data=':GET /...' --auto=none --fake -1
```
Переопределить фейковый пакет для HTTP

------
### Сборка
Для сборки понадобится: 
`make`, `gcc/clang` для Linux, `mingw` для Windows  

* Linux: `make`
* Windows: `make windows CC=x86_64-w64-mingw32-gcc`

------
### Дополнительная информация о DPI, источники идей  
* https://github.com/bol-van/zapret/blob/master/docs/readme.md  
* https://geneva.cs.umd.edu/papers/geneva_ccs19.pdf  
* https://habr.com/ru/post/335436  
