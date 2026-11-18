# Wireshark Cheat Sheet (на русском)

Практическая шпаргалка по Wireshark для анализа сетевого трафика и malware traffic analysis.

---

## Содержание

- [Горячие клавиши](#горячие-клавиши)
- [Display Filters](#display-filters)
- [Полезные фильтры по протоколам](#полезные-фильтры-по-протоколам)
- [Фильтры для поиска заражённого хоста](#фильтры-для-поиска-заражённого-хоста)
- [Statistics](#statistics)
- [Follow Stream](#follow-stream)
- [Полезные советы](#полезные-советы)
- [Фильтры под malware-анализ](#фильтры-под-malware-анализ)

---

## Горячие клавиши

| Клавиша                  | Действие                                      |
|--------------------------|-----------------------------------------------|
| `Ctrl + F`               | Поиск                                         |
| `Ctrl + E`               | Начать / остановить захват                    |
| `Ctrl + K`               | Настройки захвата                             |
| `Ctrl + Shift + L`       | Очистить фильтр                               |
| `Ctrl + /`               | Фокус на строку фильтра                       |
| `Ctrl + Alt + Shift + T` | Временные метки относительно первого пакета   |
| `g`                      | Перейти к пакету по номеру                    |
| `→` / `←`                | Следующий / предыдущий пакет                  |
| `Shift + →`              | Следующий пакет в том же потоке               |

---

## Display Filters

### Основные операторы

| Оператор     | Значение                  | Пример                              |
|--------------|---------------------------|-------------------------------------|
| `==`         | Равно                     | `ip.addr == 10.9.11.135`            |
| `!=`         | Не равно                  | `ip.addr != 10.9.11.2`              |
| `&&` / `and` | И                         | `ip.src == 10.9.11.135 && http`     |
| `\|\|` / `or`| Или                       | `http \|\| dns`                     |
| `!` / `not`  | Не                        | `!arp`                              |
| `contains`   | Содержит                  | `http.host contains "login"`        |
| `matches`    | Регулярное выражение      | `http.host matches ".*\\.ru$"`      |
| `in`         | Входит в список           | `tcp.port in {80 443 8080}`         |

---

## Полезные фильтры по протоколам

### IP и Ethernet
```wireshark
ip.addr == 10.9.11.135
ip.src == 10.9.11.135
ip.dst == 10.9.11.135
eth.addr == 08:d4:0c:7a:29:1e
```

### DNS
```wireshark
dns
dns.qry.name contains "malware"
dns.flags.response == 0          # только запросы
dns.flags.response == 1          # только ответы
```

### HTTP / HTTPS (TLS)
```wireshark
http
http.request
http.response.code == 200
http.host contains "example"
tls
tls.handshake.type == 1          # Client Hello
tls.handshake.extensions_server_name contains "suspicious"
```

### SMB / Active Directory
```wireshark
smb || smb2
kerberos
kerberos.CNameString             # имя пользователя
ntlmssp
ldap
```

### Другие полезные
```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0    # новые соединения (SYN)
tcp.analysis.retransmission                 # ретрансмиссии
icmp
arp
```

---

## Фильтры для поиска заражённого хоста

```wireshark
# Весь внешний трафик из внутренней сети
ip.src == 10.9.11.0/24 and !(ip.dst == 10.9.11.0/24)

# DNS-запросы не на Microsoft / Google / Cloudflare
dns and !(dns.qry.name contains "microsoft" or dns.qry.name contains "windows" or dns.qry.name contains "google" or dns.qry.name contains "cloudflare")

# Поиск по подозрительным доменам
dns.qry.name contains "hodkiewhcz" or tls.handshake.extensions_server_name contains "hodkiewhcz"
```

---

## Statistics

| Меню                                | Назначение                                      |
|-------------------------------------|-------------------------------------------------|
| **Statistics → Endpoints**          | Кто генерирует больше всего трафика             |
| **Statistics → Conversations**      | Кто с кем общается                              |
| **Statistics → Protocol Hierarchy** | Какие протоколы присутствуют в pcap             |
| **Statistics → DNS**                | Все DNS-запросы                                 |
| **Statistics → HTTP**               | HTTP-запросы и ответы                           |
| **Statistics → TCP Stream Graphs**  | Графики TCP-потоков                             |

---

## Follow Stream

Правой кнопкой мыши по пакету → **Follow** →:

- TCP Stream
- UDP Stream
- HTTP Stream
- TLS Stream (при наличии ключей)

Используется для чтения содержимого HTTP, команд ClickFix, C2-трафика и т.д.

---

## Полезные советы

- Всегда начинай с **Protocol Hierarchy** и **Endpoints**.
- Используй цветовую разметку: `View → Coloring Rules`.
- Можно сохранять часто используемые фильтры (иконка закладки слева от строки фильтра).
- Для очень больших pcap удобнее использовать `tshark`.
- Чтобы видеть расшифрованный HTTPS:  
  `Edit → Preferences → Protocols → TLS → (Pre)-Master-Secret log filename`

---

## Фильтры под malware-анализ

```wireshark
# Подозрительные DNS-зоны
dns.qry.name matches ".*\\.(xyz|top|icu|sbs|lol|cfd|shop)$"

# Клиентские TLS Client Hello
tls.handshake.type == 1

# Подозрительные User-Agent
http.user_agent contains "curl" or http.user_agent contains "powershell" or http.user_agent contains "wget" or http.user_agent contains "python"

# Kerberos (имя пользователя)
kerberos.CNameString

# Имя файла по SMB
smb2.filename
```

---
