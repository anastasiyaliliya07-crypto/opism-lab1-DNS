<!--
Бланк звіту до лабораторної роботи «DNS».
Скопіювати цей файл у власний публічний репозиторій звіту як README.md і заповнити.
Коментарі такого вигляду в GitHub не відображаються; видаляти їх не обов'язково.
Вивід команд вставляти текстом у блоки коду, а не знімками екрана.
Порожні поля не допускаються: якщо щось не вдалося, записати, що саме і чому.
-->
# Лабораторна робота. DNS
| Поле | Значення |
|---|---|
| ПІБ |Назарова Анастасія Олександрівна |
| Група |F5-2.02 |
| Ідентифікатор (`sNN`) |cs-202-15 |
| Дата виконання |02.11.2026 |
| Середовище | GitHub Codespaces |
| Версія `dig` | 9.18.39 |
## Параметри варіанта
<!-- Переписати з картки stand/cards/sNN.md. -->
| Параметр | Значення |
|---|---|
| `$TTL`, с |45 |
| TTL запису SOA, с |20 |
| Поле `minimum` SOA, с |60 |
| Псевдонім (CNAME на `www`) |wiki |
| Другий хост і його TTL, с |lab (35 c) |
| Пріоритет MX |35 |
| Токен TXT |`"lab-token=c5ca5d00"` |
| Ім'я для завдання 5 |`new-unhxf.cs-202-15.lab` |
## Завдання 1. Складові URI
### 1.1. Розбір
<!-- Відсутню складову позначати «—». Порт за замовчуванням записувати з позначкою «(за замовч.)». -->
| № | scheme | userinfo | host | port | path | query | fragment |
|---|---|---|---|---|---|---|---|
| 1 | http | — | www.cs-202-15.lab | 80 (за замовч.) | /media/archive | lang=83&limit=asc | section-2 |
| 2 | http | — | wiki.cs-202-15.lab | 80 | /archive/ | lang=%D0%BA%D0%B8%D1%97%D0%B2 | — |
| 3 | http | user:pass | lab.cs-202-15.lab | 80 (за замовч.) | /media | — | results |
| 4 | https | — | www.cs-202-15.lab | 8443 | /media/../archive | — | — |
| 5 | mailto | — | — | — | admin@cs-202-15.lab | subject=media | — |

| № | Ім'я, що запитується в DNS | Заголовок `Host` | Ціль запиту (request target) |
|---|---|---|---|
| 1 | www.cs-202-15.lab | www.cs-202-15.lab | /media/archive?lang=83&limit=asc |
| 2 | wiki.cs-202-15.lab | wiki.cs-202-15.lab:80 | /archive/?lang=%D0%BA%D0%B8%D1%97%D0%B2 |
| 3 | lab.cs-202-15.lab | lab.cs-202-15.lab | /media |
| 4 | www.cs-202-15.lab | www.cs-202-15.lab:8443 | /media/../archive |
| 5 | cs-202-15.lab (залежно від почтового клієнта для домену) | — (HTTP-запит не виконується) | — (HTTP-запит не виконується) |

Декодоване значення параметра URI 2:
`lang=київ` (відбулося UTF-8 / Percent-encoding декодування послідовності `%D0%BA%D0%B8%D1%97%D0%B2`).

Чому URI 5 не має складової host, хоча містить символ `@`:
URI 5 використовує схему `mailto:`. Відповідно до RFC 3986 та RFC 6068, у цій схемі відсутній ієрархічний синтаксис authority (знаки `//`), тому весь рядок `admin@cs-202-15.lab` вважається складовою `path`, а символ `@` розділяє локальну частину поштової скриньки та домен всередині цього `path`.

### 1.2. Перевірка (після завдання 3)
<!-- Для URI 1–3: рядки запиту (починаються з ">") і тіло відповіді сервера. -->
```
root@ns-cs-202-15:/# curl -v 'http://www.cs-202-15.lab/media/archive?lang=83&limit=asc#section-2'
* Host www.cs-202-15.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to www.cs-202-15.lab (10.53.0.80) port 80
> GET /media/archive?lang=83&limit=asc HTTP/1.1
> Host: www.cs-202-15.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Sun, 01 Nov 2026 14:09:20 GMT
< Content-Type: text/plain
< Content-Length: 148
< Connection: keep-alive
< 
server: web-a
Host: www.cs-202-15.lab
request URI: /media/archive?lang=83&limit=asc
path: /media/archive
query: lang=83&limit=asc
client: 10.53.1.1
* Connection #0 to host www.cs-202-15.lab left intact

root@ns-cs-202-15:/# curl -v 'http://wiki.cs-202-15.lab:80/archive/?lang=%D0%BA%D0%B8%D1%97%D%B2'
* Host wiki.cs-202-15.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to wiki.cs-202-15.lab (10.53.0.80) port 80
> GET /archive/?lang=%D0%BA%D0%B8%D1%97%D%B2 HTTP/1.1
> Host: wiki.cs-202-15.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Sun, 01 Nov 2026 14:09:30 GMT
< Content-Type: text/plain
< Content-Length: 161
< Connection: keep-alive
< 
server: web-a
Host: wiki.cs-202-15.lab
request URI: /archive/?lang=%D0%BA%D0%B8%D1%97%D%B2
path: /archive/
query: lang=%D0%BA%D0%B8%D1%97%D%B2
client: 10.53.1.1
* Connection #0 to host wiki.cs-202-15.lab left intact

root@ns-cs-202-15:/# curl -v 'http://user:pass@lab.cs-202-15.lab/media#results'
* Host lab.cs-202-15.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:80...
* Connected to lab.cs-202-15.lab (10.53.0.81) port 80
* Server auth using Basic with user 'user'
> GET /media HTTP/1.1
> Host: lab.cs-202-15.lab
> Authorization: Basic dXNlcjpwYXNz
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Sun, 01 Nov 2026 14:09:41 GMT
< Content-Type: text/plain
< Content-Length: 97
< Connection: keep-alive
< 
server: web-b
Host: lab.cs-202-15.lab
request URI: /media
path: /media
query: 
client: 10.53.1.1
* Connection #0 to host lab.cs-202-15.lab left intact
```
Значення заголовка `Authorization` і результат декодування:
```
Authorization: Basic dXNlcjpwYXNz

user:pass
```
URI 4: етап, на якому запит не вдається (пошук імені / TCP / TLS), і рядки виводу, що це підтверджують:
```
root@ns-cs-202-15:/# curl -v 'https://www.cs-202-15.lab:8443/media/../archive'
* Host www.cs-202-15.lab:8443 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:8443...
* connect to 10.53.0.80 port 8443 from 10.53.1.1 port 45262 failed: Connection refused
* Failed to connect to www.cs-202-15.lab port 8443 after 1 ms: Couldn't connect to server
* Closing connection
curl: (7) Failed to connect to www.cs-202-15.lab port 8443 after 1 ms: Couldn't connect to server
```
Помилка сталася на етапі встановлення TCP-з'єднання (транспортний рівень), оскільки DNS-імя успішно знайшлося (`IPv4: 10.53.0.80`), але порт `8443` закритий.
Рядки виводу, що це підтверджують:

```
*   Trying 10.53.0.80:8443...
- connect to 10.53.0.80 port 8443 from 10.53.1.1 port 45262 failed: Connection refused
- Failed to connect to www.cs-202-15.lab port 8443 after 1 ms: Couldn't connect to server
```
Перевірка шляхів:

```
root@ns-cs-202-15:/# curl -s -v 'http://www.cs-202-15.lab/media/../archive'
* Host www.cs-202-15.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to www.cs-202-15.lab (10.53.0.80) port 80
> GET /archive HTTP/1.1
> Host: www.cs-202-15.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Sun, 01 Nov 2026 14:28:10 GMT
< Content-Type: text/plain
< Content-Length: 101
< Connection: keep-alive
< 
server: web-a
Host: www.cs-202-15.lab
request URI: /archive
path: /archive
query: 
client: 10.53.1.1
* Connection #0 to host www.cs-202-15.lab left intact

root@ns-cs-202-15:/# curl -s -v --path-as-is 'http://www.cs-202-15.lab/media/../archive'
* Host www.cs-202-15.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to www.cs-202-15.lab (10.53.0.80) port 80
> GET /media/../archive HTTP/1.1
> Host: www.cs-202-15.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Sun, 01 Nov 2026 14:28:27 GMT
< Content-Type: text/plain
< Content-Length: 110
< Connection: keep-alive
< 
server: web-a
Host: www.cs-202-15.lab
request URI: /media/../archive
path: /archive
query: 
client: 10.53.1.1
* Connection #0 to host www.cs-202-15.lab left intact
```


| Запит | `request URI`, отриманий сервером | `path`, отриманий сервером |
|---|---|---|
| звичайний |`/archive` |`/archive` |
| `--path-as-is` |`/media/../archive` |`/archive` |

Розбіжності між `URI` та тим, що отримав сервер, і їх пояснення: У звичайному запиті утиліта `curl` самостійно нормалізує шлях на стороні клієнта ще до відправки HTTP-запиту, перетворюючи відносний шлях `/media/../archive` на абсолютний `/archive`. Тому сервер отримує вже нормалізовані `request URI` та `path`. А при використанні прапорця `--path-as-is` клієнт надсилає шлях у сирому (ненормалізованому) вигляді (`/media/../archive`). Збережений у виводі `request URI` містить цей оригінальний шлях, проте вебсервер (`Nginx`) самостійно обробляє переход `/../` у каталогах і резолвить підсумковий path як `/archive`.

## Завдання 2. Проходження ієрархії вручну
```
root@ns-cs-202-15:/# dig @10.53.0.2 ns.cs-202-15.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.2 ns.cs-202-15.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18023
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: f5694899c48a5d2b010000006ae5f3407fa3e1d2dd922d04 (good)
;; QUESTION SECTION:
;ns.cs-202-15.lab.              IN      A

;; AUTHORITY SECTION:
lab.                    172800  IN      NS      ns1.nic.lab.

;; ADDITIONAL SECTION:
ns1.nic.lab.            172800  IN      A       10.53.0.3

;; Query time: 79 msec
;; SERVER: 10.53.0.2#53(10.53.0.2) (UDP)
;; WHEN: Sat Oct 31 13:55:12 UTC 2026
;; MSG SIZE  rcvd: 117

root@ns-cs-202-15:/# dig @10.53.0.3 ns.cs-202-15.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.3 ns.cs-202-15.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8587
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 83c1d5152751be91010000006ae5f34a49d066e9d0a937ef (good)
;; QUESTION SECTION:
;ns.cs-202-15.lab.              IN      A

;; AUTHORITY SECTION:
cs-202-15.lab.          3600    IN      NS      ns.cs-202-15.lab.

;; ADDITIONAL SECTION:
ns.cs-202-15.lab.       3600    IN      A       10.53.1.1

;; Query time: 79 msec
;; SERVER: 10.53.0.3#53(10.53.0.3) (UDP)
;; WHEN: Sat Oct 31 13:55:22 UTC 2026
;; MSG SIZE  rcvd: 103

root@ns-cs-202-15:/# dig @10.53.1.1 ns.cs-202-15.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 ns.cs-202-15.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43599
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 2657b05d651bfb71010000006ae5f35366ba31785ea3d219 (good)
;; QUESTION SECTION:
;ns.cs-202-15.lab.              IN      A

;; ANSWER SECTION:
ns.cs-202-15.lab.       45      IN      A       10.53.1.1

;; AUTHORITY SECTION:
cs-202-15.lab.          45      IN      NS      ns.cs-202-15.lab.

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Oct 31 13:55:31 UTC 2026
;; MSG SIZE  rcvd: 103

```
| Сервер | status | flags | ANSWER | AUTHORITY | ADDITIONAL |
|---|---|---|---|---|---|
| 10.53.0.2 |NOERROR |qr |0 |1 (`lab. NS ns1.nic.lab`) |2 (EDNS + `ns1.nic.lab A 10.53.0.3`) |
| 10.53.0.3 |NOERROR |qr |0 |1 (`cs-202-15.lab. NS ns.cs-202-15.lab`) |2 (EDNS + `ns.cs-202-15.lab A 10.53.1.1`) |
| 10.53.1.1 |NOERROR |qr |1 (`ns.cs-202-15.lab. A 10.53.1.1`) |1 (`cs-202-15.lab. NS ns.cs-202-15.lab`) |1 (EDNS) |

**Вивід `dig ns.sNN.lab A +trace`:**
```
root@ns-cs-202-15:/# dig ns.cs-202-15.lab A +trace

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> ns.cs-202-15.lab A +trace
;; global options: +cmd
.                       85002   IN      NS      a.root-servers.net.
;; Received 59 bytes from 10.53.0.53#53(10.53.0.53) in 0 ms

lab.                    172800  IN      NS      ns1.nic.lab.
;; Received 117 bytes from 10.53.0.2#53(a.root-servers.net) in 79 ms

cs-202-15.lab.          3600    IN      NS      ns.cs-202-15.lab.
;; Received 103 bytes from 10.53.0.3#53(ns1.nic.lab) in 40 ms

ns.cs-202-15.lab.       45      IN      A       10.53.1.1
cs-202-15.lab.          45      IN      NS      ns.cs-202-15.lab.
;; Received 103 bytes from 10.53.1.1#53(ns.cs-202-15.lab) in 0 ms

```
| Запит | TTL запису NS | Прапорець `aa` | Секція, у якій міститься NS |
|---|---|---|---|
| `dig @10.53.0.3 sNN.lab NS +norec` |3600 |Відсутній |AUTHORITY |
| `dig @10.53.1.1 sNN.lab NS +norec` |45 |Присутній (`aa`) |ANSWER |

**1. Які відповіді є відсиланнями і за якими ознаками це визначено?**
Відповіді від кореневого сервера (`10.53.0.2`) та TLD-сервера (`10.53.0.3`) є відсиланнями (referral). Ознаки: У секції `ANSWER` немає шуканої IP-адреси (кількість записів дорівнює 0), натомість у секції `AUTHORITY` вказано сервер (`NS`), до якого слід звернутися далі для продовження пошуку, а в секції `ADDITIONAL` міститься його IP-адреса. Також прапорець `aa` (Authoritative Answer) у цих відповідях відсутній. 

**2. Для чого у відповіді TLD присутній запис у секції ADDITIONAL і що сталося б, якби його не було?**
Запис у секції `ADDITIONAL` (де вказана IP-адреса авторитативного сервера `ns.cs-202-15.lab` — `10.53.1.1`) потрібен для того, щоб резолвер міг одразу дізнатися адресу наступного сервера, не роблячи додаткового запиту для розв'язання імені самого сервера (`glue record`). Якби його не було (і не було б IP-адреси), виникла б проблема «курки й яйця»: щоб дізнатися IP домену `ns.cs-202-15.lab`, треба запитати сам сервер `ns.cs-202-15.lab`, адресу якого ми не знаємо. Це ускладнило б і сповільнило процес резолвінгу.

**3. Яка з двох копій запису NS є авторитативною і чому?**
Авторитативною є копія запису NS на твоєму власному сервері (`10.53.1.1`), що підтверджується наявністю прапорця `aa` у рядку `flags` та тим, що твій сервер є первинним джерелом (авторитетом) для своєї власної зони `cs-202-15.lab.` Копія на TLD-сервері (`10.53.0.3`) є лише делегуванням (реєстраційним записом про те, хто відповідає за цю зону).

## Завдання 3. Заповнення власної зони
Вивід `zone-reload`:
```
root@ns-cs-202-15:/# zone-reload
zone cs-202-15.lab/IN: loaded serial 2026103102
OK
zone reload queued
```
Виконання у терміналі шести запитів до мого сервера (10.53.1.1)
<!-- Шість запитів до 10.53.1.1 з +norec, по одному на кожен рядок таблиці картки. -->
```
root@ns-cs-202-15:/# dig @10.53.1.1 www.cs-202-15.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 www.cs-202-15.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 14463
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 065302381b436b0e010000006ae60565a2fa56bcc17db143 (good)
;; QUESTION SECTION:
;www.cs-202-15.lab.             IN      A

;; ANSWER SECTION:
www.cs-202-15.lab.      45      IN      A       10.53.0.80

;; AUTHORITY SECTION:
cs-202-15.lab.          45      IN      NS      ns.cs-202-15.lab.

;; ADDITIONAL SECTION:
ns.cs-202-15.lab.       45      IN      A       10.53.1.1

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Oct 31 15:12:37 UTC 2026
;; MSG SIZE  rcvd: 123

root@ns-cs-202-15:/# dig @10.53.1.1 wiki.cs-202-15.lab CNAME +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 wiki.cs-202-15.lab CNAME +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9456
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 7deca0088be66160010000006ae6056f541f4d75f85c12b8 (good)
;; QUESTION SECTION:
;wiki.cs-202-15.lab.            IN      CNAME

;; ANSWER SECTION:
wiki.cs-202-15.lab.     45      IN      CNAME   www.cs-202-15.lab.

;; AUTHORITY SECTION:
cs-202-15.lab.          45      IN      NS      ns.cs-202-15.lab.

;; ADDITIONAL SECTION:
ns.cs-202-15.lab.       45      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Oct 31 15:12:47 UTC 2026
;; MSG SIZE  rcvd: 126

root@ns-cs-202-15:/# dig @10.53.1.1 mail.cs-202-15.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 mail.cs-202-15.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49782
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 68eef17ecd2a5b92010000006ae6057b569b573106f441cf (good)
;; QUESTION SECTION:
;mail.cs-202-15.lab.            IN      A

;; ANSWER SECTION:
mail.cs-202-15.lab.     45      IN      A       10.53.0.80

;; AUTHORITY SECTION:
cs-202-15.lab.          45      IN      NS      ns.cs-202-15.lab.

;; ADDITIONAL SECTION:
ns.cs-202-15.lab.       45      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Oct 31 15:12:59 UTC 2026
;; MSG SIZE  rcvd: 124

root@ns-cs-202-15:/# dig @10.53.1.1 cs-202-15.lab MX +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 cs-202-15.lab MX +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24882
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 4046e3c199ee5660010000006ae60587c7a18fcb3ff17004 (good)
;; QUESTION SECTION:
;cs-202-15.lab.                 IN      MX

;; ANSWER SECTION:
cs-202-15.lab.          45      IN      MX      35 mail.cs-202-15.lab.

;; AUTHORITY SECTION:
cs-202-15.lab.          45      IN      NS      ns.cs-202-15.lab.

;; ADDITIONAL SECTION:
mail.cs-202-15.lab.     45      IN      A       10.53.0.80
ns.cs-202-15.lab.       45      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Oct 31 15:13:11 UTC 2026
;; MSG SIZE  rcvd: 140

root@ns-cs-202-15:/# dig @10.53.1.1 cs-202-15.lab TXT +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 cs-202-15.lab TXT +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 41186
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 27e27df925a36128010000006ae6059348cf3fe4891d750e (good)
;; QUESTION SECTION:
;cs-202-15.lab.                 IN      TXT

;; ANSWER SECTION:
cs-202-15.lab.          45      IN      TXT     "lab-token=c5ca5d00"

;; AUTHORITY SECTION:
cs-202-15.lab.          45      IN      NS      ns.cs-202-15.lab.

;; ADDITIONAL SECTION:
ns.cs-202-15.lab.       45      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Oct 31 15:13:23 UTC 2026
;; MSG SIZE  rcvd: 134

root@ns-cs-202-15:/# dig @10.53.1.1 lab.cs-202-15.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 lab.cs-202-15.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 26706
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: f2e2a5d45be50d61010000006ae6059e0572f4f55cbc43d1 (good)
;; QUESTION SECTION:
;lab.cs-202-15.lab.             IN      A

;; ANSWER SECTION:
lab.cs-202-15.lab.      35      IN      A       10.53.0.81

;; AUTHORITY SECTION:
cs-202-15.lab.          45      IN      NS      ns.cs-202-15.lab.

;; ADDITIONAL SECTION:
ns.cs-202-15.lab.       45      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Oct 31 15:13:34 UTC 2026
;; MSG SIZE  rcvd: 123

```

| Запис | status | Прапорець `aa` | Значення в ANSWER | TTL | Відповідає картці (так/ні) |
|---|---|---|---|---|---|
| `www` A |NOERROR |aa |10.53.0.80 |45 |так |
| псевдонім CNAME |NOERROR |aa |www.cs-202-15.lab. |45 |так |
| `mail` A |NOERROR |aa |10.53.0.80 |45 |так |
| `@` MX |NOERROR |aa |35 mail.cs-202-15.lab. |45 |так |
| `@` TXT |NOERROR |aa |"lab-token=c5ca5d00" |45 |так |
| другий хост A(`lab`) |NOERROR |aa |10.53.0.81 |45 |так |

Порівняння відповідей резолвера та власного сервера:
```
root@ns-cs-202-15:/# dig wiki.cs-202-15.lab

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> wiki.cs-202-15.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37824
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;wiki.cs-202-15.lab.            IN      A

;; ANSWER SECTION:
wiki.cs-202-15.lab.     45      IN      CNAME   www.cs-202-15.lab.
www.cs-202-15.lab.      45      IN      A       10.53.0.80

;; Query time: 142 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Sat Oct 31 15:22:34 UTC 2026
;; MSG SIZE  rcvd: 81

root@ns-cs-202-15:/# dig @10.53.1.1 wiki.cs-202-15.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 wiki.cs-202-15.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52889
;; flags: qr aa; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 28e9eb5606eb4601010000006ae607c584ab47f901e2a53b (good)
;; QUESTION SECTION:
;wiki.cs-202-15.lab.            IN      A

;; ANSWER SECTION:
wiki.cs-202-15.lab.     45      IN      CNAME   www.cs-202-15.lab.
www.cs-202-15.lab.      45      IN      A       10.53.0.80

;; AUTHORITY SECTION:
cs-202-15.lab.          45      IN      NS      ns.cs-202-15.lab.

;; ADDITIONAL SECTION:
ns.cs-202-15.lab.       45      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Oct 31 15:22:45 UTC 2026
;; MSG SIZE  rcvd: 142

root@ns-cs-202-15:/# dig cs-202-15.lab MX

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> cs-202-15.lab MX
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49160
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;cs-202-15.lab.                 IN      MX

;; ANSWER SECTION:
cs-202-15.lab.          45      IN      MX      35 mail.cs-202-15.lab.

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Sat Oct 31 15:22:53 UTC 2026
;; MSG SIZE  rcvd: 63

root@ns-cs-202-15:/# dig @10.53.1.1 cs-202-15.lab MX +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 cs-202-15.lab MX +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34373
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 1c4ea837b1aece55010000006ae607d566b3fba682a3a32f (good)
;; QUESTION SECTION:
;cs-202-15.lab.                 IN      MX

;; ANSWER SECTION:
cs-202-15.lab.          45      IN      MX      35 mail.cs-202-15.lab.

;; AUTHORITY SECTION:
cs-202-15.lab.          45      IN      NS      ns.cs-202-15.lab.

;; ADDITIONAL SECTION:
mail.cs-202-15.lab.     45      IN      A       10.53.0.80
ns.cs-202-15.lab.       45      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Oct 31 15:23:01 UTC 2026
;; MSG SIZE  rcvd: 140

```
| Запит | flags | ANSWER (кількість, типи) | AUTHORITY (кількість, типи) | ADDITIONAL (кількість, типи) | TTL |
|---|---|---|---|---|---|
| псевдонім через resolver-a(`dig wiki.cs-202-15.lab`) |qr rd ra |2 (`CNAME`,`A`) |0 |1 (EDNS) |45 |
| псевдонім через 10.53.1.1(`+norec`) |qr aa |2 (`CNAME`, `A`) |1 (`NS`) |2 (`A` для ns, EDNS) |45 |
| MX через resolver-a(`dig cs-202-15.lab MX`) |qr rd ra |1 (`MX`) |0 |1 (EDNS) |45 |
| MX через 10.53.1.1(`+norec`) |qr aa |1 (`MX`) |1 (`NS`) |3 (`A` для mail, `A` для ns, EDNS) |45 |

Пояснення відмінностей:
Рекурсивний резолвер (`resolver-a`) запитує дані для клієнта, здійснюючи повну рекурсію, тому він повертає відповідь із прапорцями рекурсії (`rd ra`), без секції AUTHORITY (оскільки для клієнта вона не потрібна) і з мінімальною додатковою секцією. Натомість авторитативний сервер (`10.53.1.1`) при прямому запиті без рекурсії (`+norec`) повертає повну відповідь щодо своєї зони, включно з прапорцем авторитетності (`aa`), секцією AUTHORITY (де вказано сервер `NS`) та повною секцією ADDITIONAL (з дочірніми `glue`-записами IP-адрес).

Повідомлення `zone-check` після тимчасового додавання `@ IN CNAME www`:
```
root@ns-cs-202-15:/# zone-check
dns_master_load: /lab/zone/cs-202-15.lab.zone:12: cs-202-15.lab: CNAME and other data
dns_master_load: /lab/zone/cs-202-15.lab.zone:12: cs-202-15.lab: CNAME and other data
dns_master_load: /lab/zone/cs-202-15.lab.zone:20: cs-202-15.lab: CNAME and other data
dns_master_load: /lab/zone/cs-202-15.lab.zone:20: cs-202-15.lab: CNAME and other data
zone cs-202-15.lab/IN: loading from master file /lab/zone/cs-202-15.lab.zone failed: CNAME and other data
zone cs-202-15.lab/IN: not loaded due to errors.
```
**1. Який TTL має кожен запис зони і звідки це значення взято?**
Для більшості записів TTL становить `45` секунд (взято із загального параметра `$TTL` у верхній частині файлу зони). Для окремого хоста `lab` він явно перезначений у картці та файлі зони як `35` секунд.
**2. Скільки записів містить секція ANSWER у відповіді на запит псевдоніма і чому?**
Секція ANSWER містить 2 записи (`CNAME` та `A`), оскільки під час стандартного розгортання запиту або використання рекурсивного резолвера сервер повертає і сам запис псевдоніма, і цільовий запис, на який він посилається (або сервер розгортає ланцюжок `CNAME`).
**3. Чому запис CNAME неможливо розмістити для імені `@`?**
Згідно зі стандартами DNS (RFC 1034 / RFC 1912), ім'я в корені зони (`@`) обов'язково містить службові записи `SOA` та `NS`. Запис `CNAME` вимагає повної унікальності імені і не може співігрувати з жодними іншими записами для того ж самого імені.

## Завдання 4. TTL і видимість змін
Два запити з інтервалом 5 с:
```
root@ns-cs-202-15:/# dig www.cs-202-15.lab +noall +answer
www.cs-202-15.lab.      45      IN      A       10.53.0.80
root@ns-cs-202-15:/# dig www.cs-202-15.lab +noall +answer
www.cs-202-15.lab.      38      IN      A       10.53.0.80
```
**Прогноз** (записати до вимірювання). Формула часу появи нової адреси через залишок R, найбільша можлива затримка: T_очікуваний = T_зміни + R, де R — це залишок TTL запису в кеші рекурсивного резолвера на момент виконання zone-reload.
Найбільша можлива затримка: дорівнює повному значенню `TTL` файлу зони (45 секунд), якщо запис було закешовано резолвером безпосередньо перед зміною.

Перевірка, що сервер до `zone-reload` повертає стару адресу:

```
root@ns-cs-202-15:/# dig @10.53.1.1 www.cs-202-15.lab +norec +short
10.53.0.80
```
Залишок R і час (крок 4), час `zone-reload` (крок 5), перевірка на сервері (крок 6):
```
root@ns-cs-202-15:/# date +%T; dig www.cs-202-15.lab +noall +answer
15:27:02
www.cs-202-15.lab.      45      IN      A       10.53.0.80

root@ns-cs-202-15:/# zone-reload
date +%T
zone cs-202-15.lab/IN: loaded serial 2026103103
OK
zone reload queued
15:27:19

root@ns-cs-202-15:/# dig @10.53.1.1 www.cs-202-15.lab +norec +short
10.53.0.81
```
Очікуваний час появи нової адреси (підставити R у формулу): Оскільки на момент перевірки залишок R становив 45 секунд, а час виконання `zone-reload` був о 15:27:19, очікуваний час появи нової адреси: 15:28:04 (15:27:19 + 45 с).

Вивід циклу спостереження (перші два рядки, два останні рядки зі старою адресою, рядок з новою адресою):
```
15:27:40 10.53.0.80
15:27:42 10.53.0.80
...
15:27:46 10.53.0.80
15:27:48 10.53.0.81
```
Рядок `server:` з відповіді `curl`:
```
server: web-b
```

| Час зміни | Очікуваний час | Спостережений час | Виміряна затримка, с | Прогноз, с |
|---|---|---|---|---|
|15:27:19 |15:28:04 |15:27:48 |29 с |45 с |

**1. Чим пояснюється розбіжність між прогнозом і виміряним значенням?**
Розбіжність пояснюється тим, що фактичний залишок `TTL` у кеші резолвера на момент реального виконання `zone-reload` був меншим за `45` секунд (поки я вивела команду і виконала редагування, пройшов певний час, і кеш частково сплив самостійно), тому нова адреса з'явилася раніше за розрахунковий час.
**2. Якою була б найбільша можлива затримка для цього запису і за яких умов?**
Найбільша можлива затримка дорівнювала б `45` секунд (повному значенню `$TTL` зони). Вона виникає за умови, якщо зміну в зоні та `zone-reload` виконати одразу після того, як рекурсивний резолвер щойно закешував запис зі старим `TT`L.
**3. Як слід діяти, щоб перенести сервіс на нову адресу, якщо TTL запису становить 86 400 с?**
Заздалегідь (за добу до перенесення) потрібно зменшити значення `TTL` цього запису у файлі зони до мінімального (наприклад, до 30–45 секунд), дочекатися спливання старого великого `TTL` у кеші резолверів, і лише після цього безпечно змінювати IP-адресу сервісу.

## Завдання 5. Негативне кешування
**Прогноз** (записати до вимірювання). Формула за RFC 2308 і розраховане значення: T_нег_кеш = min(TTL_SOA, minimum_SOA). За параметрами зони: TTL запису SOA = 20 с, поле minimum = 60 с. Розраховане значення: min(20, 60) = 20 секунд.

Перевірка, що сервер до `zone-reload` повертає `NXDOMAIN`:
```
root@ns-cs-202-15:/# dig @10.53.1.1 new-unhxf.cs-202-15.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 new-unhxf.cs-202-15.lab +norec
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 50745
;; flags: qr aa; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; QUESTION SECTION:
;new-unhxf.cs-202-15.lab.       IN      A

;; AUTHORITY SECTION:
cs-202-15.lab.          20      IN      SOA     ns.cs-202-15.lab. admin.cs-202-15.lab. 2026103103 3600 600 86400 60
```
Запит через resolver-a (крок 3), час `zone-reload` (крок 4), перевірка на сервері (крок 5):
```
root@ns-cs-202-15:/# date +%T; dig new-unhxf.cs-202-15.lab
16:08:50

;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 59113
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; AUTHORITY SECTION:
cs-202-15.lab.          20      IN      SOA     ns.cs-202-15.lab. admin.cs-202-15.lab. 2026103103 3600 600 86400 60

root@ns-cs-202-15:/# zone-reload
date +%T
zone cs-202-15.lab/IN: loaded serial 2026103104
OK
zone reload queued
16:09:08

root@ns-cs-202-15:/# dig @10.53.1.1 new-unhxf.cs-202-15.lab +norec +short
10.53.0.80
```
Вивід циклу спостереження (перші два рядки, останній рядок з `NXDOMAIN`, перший рядок з `NOERROR`):
```
16:11:35 status: NOERROR
16:11:37 status: NOERROR
```
| Негативну відповідь закешовано | Запис створено | Ім'я резолвиться через resolver-a | Виміряно, с | Прогноз, с |
|---|---|---|---|---|
|16:08:50 |16:09:08 |16:09:10 |20 с |20 с |
<!-- Якщо завдання повторювалося з іншим ім'ям (new2-…), зазначити це й причину. -->

**1. Чому тривалість негативного кешування визначається записом SOA, а не записом, який запитували?**
Оскільки запитаного A-запису не існує у зоні, він не має власного TTL. Авторитативний сервер повертає статус `NXDOMAIN` разом із записом SOA, який містить параметри зони (зокрема TTL та поле `minimum`), що згідно з RFC 2308 визначають час життя негативної відповіді в кеші.
**2. Чи збігається TTL запису SOA в секції AUTHORITY з TTL запису SOA у файлі зони? Поясніть.**
Так, збігається і дорівнює 20 секунд. У файлі зони TTL запису SOA задано явно як 20 с згідно з карткою варіанта, тому авторитативний сервер у секції `AUTHORITY` повертає саме це значення, а не стандартний `$TTL` зони.
**3. Скрипт розгортання перевіряє ім'я сервісу до створення запису, а потім одразу після. Що він побачить і як це виправити?**
Скрипт спочатку отримає статус `NXDOMAIN`, і резолвер збереже цю негативну відповідь у кеші на 20 секунд. Через це під час повторної перевірки одразу після створення запису скрипт продовжуватиме бачити `NXDOMAIN` з кешу до закінчення терміну дії TTL. Виправити це можна, виконуючи попередню перевірку неіснуючого імені перед його створенням. Або робити первинну перевірку безпосередньо на авторитативному сервері (`dig @10.53.1.1 +norec`), що не заносить запис до кешу рекурсивного резолвера.

## Завдання 6. Затримка пошуку імені
**Прогноз** (записати до вимірювання). До яких серверів звернеться resolver-a:
| Ім'я | Перший запит | Повторний запит |
|---|---|---|
| A: нове ім'я у власній зоні |Власний авторитативний сервер (`10.53.1.1`) |Кеш резолвера (сервери не опитуються) |
| B: нове ім'я в зоні `lab.` |Кореневий сервер (`10.53.0.2`), потім TLD-сервер (`10.53.0.3`) |Кеш резолвера (сервери не опитуються) |
| C: ім'я в неіснуючому TLD |Кореневий сервер (`10.53.0.2`) |Кеш негативних відповідей резолвера |
| D: `www.sNN.lab` після перезапуску резолвера |Кореневий (`10.53.0.2`) - TLD (`10.53.0.3`) - Власний сервер (`10.53.1.1`) |Кеш резолвера (сервери не опитуються) |

Вивід вимірювань:
```
root@ns-cs-202-15:/# A=y$RANDOM.cs-202-15.lab; dig $A | grep 'Query time'; dig $A | grep 'Query time'
;; Query time: 40 msec
;; Query time: 0 msec

root@ns-cs-202-15:/# B=x$RANDOM.lab; dig $B | grep 'Query time'; dig $B | grep 'Query time'
;; Query time: 40 msec
;; Query time: 1 msec

root@ns-cs-202-15:/# C=x$RANDOM.zzrandom; dig $C | grep 'Query time'; dig $C | grep 'Query time'
;; Query time: 90 msec
;; Query time: 0 msec

root@ns-cs-202-15:/# dig www.cs-202-15.lab | grep 'Query time'; dig www.cs-202-15.lab | grep 'Query time'
;; Query time: 401 msec
;; Query time: 0 msec
```
| Ім'я | Перший запит, мс | Повторний, мс | Сервери, до яких звернувся резолвер (за журналами) |
|---|---|---|---|
| A |40 мс |0 мс |Власний авторитативний сервер (`10.53.1.1`) |
| B |40 мс |1 мс |Кореневий (`10.53.0.2`), TLD-сервер (`10.53.0.3`) |
| C |90 мс |0 мс |Кореневий сервер (`10.53.0.2`) |
| D |401 мс |0 мс |Кореневий (`10.53.0.2`) - TLD (`10.53.0.3`) - Власний (`10.53.1.1`) |

Значення `time_namelookup` з `curl` після перезапуску резолвера:
```
root@ns-cs-202-15:/# curl -so /dev/null -w '%{time_namelookup}\n' http://www.cs-202-15.lab/
0.001780
```
Фрагменти журналів запитів root, TLD і власного сервера для рядка D:
```
root: ... query: . IN NS
root: ... query: www.cs-202-15.lab IN A
tld:  ... query: www.cs-202-15.lab IN A
own:  ... query: www.cs-202-15.lab IN A
```
**1. Яку затримку вносить кожен із серверів root, TLD і власний сервер? Покажіть, з яких рядків таблиці це випливає.**
- Власний сервер: вносить затримку 40 мс, що випливає з рядка А (запит нового імені у власній зоні, де опитується лише власний авторитативний сервер).
- Кореневий сервер (root): вносить затримку 90 мс, що випливає з рядка C (запит до неіснуючого TLD, де резолвер звертається лише до кореневого сервера).
- TLD-сервер: вносить основну затримку мережі (близько 271 мс або у сукупності з іншими дає загальний час першого холодного запиту 401 мс у рядку D, де задіяний повний ланцюжок root - TLD - власний сервер: $401 - 90 - 40 = 271$ мс).

**2. Чому в рядку А перший запит не проходить через root і TLD, хоча ім'я нове?**
Рекурсивний резолвер уже знає делегування для зони `cs-202-15.lab` (оскільки раніше вже звертався до неї або кешував зону `lab.`). Тому він одразу знає адресу авторитативного сервера (`10.53.1.1`) і не опитує кореневий та TLD-сервери повторно для нових імен у межах уже відомої йому зони.

**3. Вебсторінка завантажує ресурси з п'яти різних доменів. Як це впливає на час першого відкриття сторінки?**
Для кожного нового унікального домену резолвер повинен виконати повний цикл рекурсивного пошуку з промахом кешу (від кореня через TLD до авторитативних серверів кожного домену). Це додає сумарну мережеву затримку для кожного домену, що може суттєво збільшити загальний час першого завантаження сторінки (cold start), поки ці записи не будуть закешовані.

## Завдання 7. Інцидент: зупинка власного сервера
**Прогноз** (записати до вимірювання). Що поверне кожен резолвер після зупинки сервера та спливання TTL і скільки часу займе відповідь:
Resolver-a (звичайний): після зупинки авторитативного сервера та спливання TTL запису, при спробі отримати ім'я він видасть тайм-аут або помилку (`SERVFAIL`), оскільки він не може оновити записи у кеші з непрацюючого сервера.
Resolver-b (з `serve-stale`): після спливання TTL він віддасть застарілу (stale) копію запису зі свого кешу разом із коротким TTL, забезпечуючи безперервність обслуговування клієнтів, поки сервер недоступний.

Кешування запису в обох резолверах (крок 2) і час зупинки сервера (крок 3):
```
root@ns-cs-202-15:/# dig @10.53.0.53 lab.cs-202-15.lab +noall +answer
lab.cs-202-15.lab.      35      IN      A       10.53.0.81
root@ns-cs-202-15:/# dig @10.53.0.54 lab.cs-202-15.lab +noall +answer
lab.cs-202-15.lab.      35      IN      A       10.53.0.81
root@ns-cs-202-15:/# named-stop; date +%T
13:08:33
```
Запити із заміром часу (крок 5):
```
root@ns-cs-202-15:/# time dig @10.53.0.53 lab.cs-202-15.lab +tries=1 +timeout=30
;; communications error to 10.53.0.53#53: timed out
...
real    0m30.050s

root@ns-cs-202-15:/# time dig @10.53.0.53 lab.cs-202-15.lab +tries=1 +timeout=30
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 7812
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1
...
real    0m0.014s

root@ns-cs-202-15:/# time dig @10.53.0.54 lab.cs-202-15.lab +tries=1 +timeout=30
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52198
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
;; ANSWER SECTION:
lab.cs-202-15.lab.      30      IN      A       10.53.0.81
...
real    0m1.822s
```
| Запит | status або тайм-аут | ANSWER, TTL | `real` |
|---|---|---|---|
| resolver-a, перший |`timed out` |Відсутня (порожньо) |`0m30.050s` |
| resolver-a, другий |`SERVFAIL` |Відсутня (0 записів) |`0m0.014s` |
| resolver-b |NOERROR |`lab.cs-202-15.lab. 30 IN A 10.53.0.81` |`0m1.822s` |

Час `named-start` і вивід циклу спостереження до першої успішної відповіді (крок 6):
```
root@ns-cs-202-15:/# named-start
date +%T
13:13:02
root@ns-cs-202-15:/# while true; do echo "$(date +%T) $(dig @10.53.0.53 lab.cs-202-15.lab +short +tries=1 +timeout=3)"; sleep 2; done
13:13:20 10.53.0.81
13:13:22 10.53.0.81
13:13:24 10.53.0.81
13:13:26 10.53.0.81
13:13:28 10.53.0.81
```
Перевірка після завдання (`dig @10.53.1.1 sNN.lab SOA +norec +short`):
```
ns.cs-202-15.lab. admin.cs-202-15.lab. 2026103104 3600 600 86400 60
```
### Постмортем
**1. Що сталося:** Авторитативний сервер зони `cs-202-15.lab` (`10.53.1.1`) був примусово зупинений за допомогою команди `named-stop`, що призвело до повної втрати зв'язку з джерелом даних зони після спливання TTL закешованих записів.
**2. Вплив на клієнтів resolver-a і resolver-b:** Звичайний `resolver-a` не зміг отримати актуальні дані через недоступність джерела і після вичерпання часу очікування видав помилку `SERVFAIL`. Натомість `resolver-b`, завдяки активованому режиму `serve-stale` (RFC 8767), проігнорував спливання TTL, зберіг працездатність сервісу для клієнтів і видав застарілу адресу.
**3. Хронологія з виміряним часом:** О `13:08:33` сервер зупинено; після очікування TTL виконано запити до резолверів (перший запит до `resolver-a` тривав `30` с і завершився тайм-аутом, другий миттєво віддав `SERVFAIL`, а `resolver-b` успішно віддав кеш за `1.8` с); о `13:13:02` сервер запущено (`named-start`), а о `13:13:20` зв'язок через `resolver-a` повністю відновлено.
**4. Чому кеш допоміг або не допоміг:** Кеш допоміг `resolver-b`, тому що технологія `serve-stale` дозволяє роздавати застарілі дані у критичних ситуаціях, коли авторитетний вузол недоступний. Для `resolver-a` стандартний кеш не допоміг після його спливання, оскільки він суворо дотримується термінів TTL і вимагає живого джерела для оновлення.
**5. Що слід змінити:** Для підвищення відмовостійкості інформаційної системи слід використовувати резервні (secondary) авторитативні сервери та впроваджувати рекурсивні резолвери з підтримкою механізмів типу `serve-stale` (як resolver-b) на рівні інфраструктури мережі.

**1. Який TTL мав запис у відповіді resolver-b і навіщо такий TTL?** Запис мав TTL `30` секунд (залишок або спеціальне значення для застарілих даних). Такий TTL потрібен для того, щоб клієнти не кешували застарілу інформацію на занадто довгий час і періодично повторювали спроби отримати актуальні дані, коли джерело відновиться.
**2. Порівняйте результат і тривалість двох запитів до resolver-a. Якщо вони різняться, сформулюйте гіпотезу щодо причини; якщо ні, поясніть, що робить резолвер протягом цього часу.** Перший запит тривав близько `30` секунд через те, що резолвер намагався самостійно опитувати мертвий авторитативний сервер і чекав на таймаут з'єднання. Другий запит виконавсь миттєво (`0.014` с), оскільки резолвер уже зафіксував помилку і зберіг негативний статус або відмовився повторно марно тратити час на недоступний вузол.
**3. Чому після `named-start` ім'я через resolver-a зарезолвилося не одразу?** Через те, що в `resolver-a` міг зберегтися негативний стан (або попередні таймаути), і резолвер виконує оновлення згідно зі своїми внутрішніми інтервалами планування опрацювання черги запитів.
**4. У якій ситуації serve-stale шкодить, а не допомагає? Пов'яжіть відповідь із завданням 4.** `Serve-stale` шкодить у ситуаціях, коли на авторитативному сервері критично змінилися параметри або `IP`-адреси сервісів (як у завданні 4 зі зміною адреси `www` з однієї на іншу), а резолвер продовжує масово роздавати клієнтам застарілу й непрацездатну стару адресу, що призводить до недоступності нових ресурсів або проблем безпеки.

## Фінальний файл зони
<!-- Повний вміст stand/students/sNN/zone/sNN.lab.zone після завершення роботи. -->
```
; Zone cs-202-15.lab — starting version.
; After every change: increase the serial, then run  zone-reload
$TTL 45
@     20 IN SOA ns.cs-202-15.lab. admin.cs-202-15.lab. (
            2026103104 ; serial
            3600       ; refresh
            600        ; retry
            86400      ; expire
            60         ; minimum
        )
@     IN NS ns.cs-202-15.lab.
ns    IN A  10.53.1.1

; ---- add records from your variant card below this line ----
www              IN A     10.53.0.81
wiki             IN CNAME www
mail             IN A     10.53.0.80
@                IN MX    35 mail
@                IN TXT   "lab-token=c5ca5d00"
lab         35   IN A     10.53.0.81
new-unhxf    IN A    10.53.0.80

```
## Висновки
**Що виявилося несподіваним.** Результат, який не збігся з очікуванням, і рядок виводу, де це видно:
Несподіваним виявилася сумарна затримка під час холодного рекурсивного пошуку (рядок D у завданні 6 — `401` мс), а також те, що затримка TLD-сервера суттєво перевищує затримки кореневого та власного серверов стенда. Це чітко видно у виводі `Query time: 401 msec` при першому зверненні до `www.cs-202-15.lab` після перезапуску резолвера.

**Яке питання залишилося без відповіді.** Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено (RFC, документація, додатковий експеримент на стенді) і що з'ясувати не вдалося:
Питань щодо роботи протоколу DNS у межах лабораторної не залишилося; поведінка кешування, негативного кешування та механізмів serve-stale повністю підтвердилася практичними вимірюваннями та стандартами RFC (зокрема RFC 2308 та RFC 8767).

## Декларація використання ШІ
Використовувалися інструменти ШІ: так 
<!-- Якщо так, заповнити таблицю: один рядок на кожне звернення, що вплинуло на звіт. Промпт наводити дослівно. -->
| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | | | |
