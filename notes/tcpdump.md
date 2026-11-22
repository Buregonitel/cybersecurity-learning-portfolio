# Шпаргалка по `tcpdump`

Быстрый справочник по захвату, фильтрации и анализу сетевого трафика в консоли для задач диагностики сети и веб-безопасности.

---

## 1. Базовый запуск и управление выводом

```bash
# Список доступных сетевых интерфейсов
sudo tcpdump -D

# Захват трафика на конкретном интерфейсе (например, lo, eth0, enp4s0, any)
sudo tcpdump -i lo

# Не преобразовывать IP в доменные имена и порты в названия сервисов
sudo tcpdump -i lo -nn

# Остановить захват после N пакетов
sudo tcpdump -i lo -nn -c 20

# Записать сырые пакеты в .pcap-файл
sudo tcpdump -i lo -nn -w capture.pcap

# Прочитать сохранённый .pcap-файл
sudo tcpdump -nn -r capture.pcap
```

---

## 2. Детализация и формат вывода

```bash
# Показать полезную нагрузку в ASCII (удобно для открытого HTTP и текстовых протоколов)
sudo tcpdump -i lo -nn -A 'tcp port 8080'

# Показать пакеты в HEX и ASCII
sudo tcpdump -i lo -nn -X 'tcp port 8080'

# Подробный вывод заголовков (-v, -vv, -vvv)
sudo tcpdump -i any -nn -vv 'udp port 53 or tcp port 53'

# Убрать буферизацию вывода при передаче в другие утилиты (-l)
sudo tcpdump -i lo -nn -l 'tcp port 8080' | grep "GET "
```

---

## 3. BPF-фильтры

```bash
# По IP-адресу (источник или назначение)
sudo tcpdump -i any -nn host 192.0.2.20

# Только отправитель (src) или только получатель (dst)
sudo tcpdump -i lo -nn src 127.0.0.1
sudo tcpdump -i lo -nn dst 127.0.0.1

# По порту и протоколу (tcp, udp, icmp)
sudo tcpdump -i any -nn tcp port 80 or tcp port 443
sudo tcpdump -i any -nn udp port 53
sudo tcpdump -i any -nn icmp

# По подсети
sudo tcpdump -i any -nn net 192.168.1.0/24

# Комбинирование условий (and, or, not, скобки)
sudo tcpdump -i any -nn 'host 192.168.1.45 and (tcp port 80 or tcp port 443) and not port 22'
```

---

## 4. Частые сценарии диагностики

```bash
# Только управляющие TCP-флаги (SYN, RST, FIN) — рукопожатие, закрытие и обрывы
sudo tcpdump -i lo -nn 'tcp port 8080 and (tcp[tcpflags] & (tcp-syn|tcp-rst|tcp-fin) != 0)'

# Только SYN-пакеты (поиск попыток подключений и ретрансмитов при таймауте)
sudo tcpdump -i any -nn 'tcp port 8080 and (tcp[tcpflags] & tcp-syn != 0)'

# DNS-запросы и ответы (UDP и TCP порт 53)
sudo tcpdump -i any -nn 'udp port 53 or tcp port 53'

# Фильтрация пакетов с ненулевой полезной нагрузкой TCP (только пакеты с данными)
sudo tcpdump -nn -r capture.pcap 'tcp port 8080 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```

---

## 5. Как быстро читать строку TCP

`18:41:00.193816 IP 127.0.0.1.45118 > 127.0.0.1.8080: Flags [P.], seq 1:85, ack 1, win 512, length 84`

* **`127.0.0.1.45118 > 127.0.0.1.8080`** — `IP.порт` источника и `IP.порт` назначения.
* **`Flags [...]`** — TCP-флаги:
  * `[S]` — `SYN` (запрос на установку соединения)
  * `[S.]` — `SYN, ACK` (подтверждение запроса на соединение)
  * `[.]` — `ACK` (подтверждение получения данных)
  * `[P.]` — `PSH, ACK` (передача данных прикладному уровню)
  * `[F.]` — `FIN, ACK` (корректное завершение соединения)
  * `[R]` / `[R.]` — `RST` (мгновенный сброс соединения, часто при закрытом порте)
* **`seq` / `ack`** — порядковый номер и номер подтверждения.
* **`length`** — длина полезной нагрузки в байтах (`0` — пакет без данных, например `SYN` или чистый `ACK`).

---

## 6. Связка `tcpdump` + Wireshark

```bash
# 1. Захватить нужный трафик в файл
sudo tcpdump -i lo -nn -w local_http.pcap 'tcp port 8080'

# 2. Открыть сохранённый дамп в Wireshark для глубокого анализа
wireshark local_http.pcap
```