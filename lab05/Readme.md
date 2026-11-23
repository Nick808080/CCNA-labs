# Статическая маршрутизация (Static Routing) в Cisco IOS

Лабораторная работа по настройке статической маршрутизации между тремя маршрутизаторами Cisco IOS: настройка интерфейсов, параметров IPv4 и шлюзов по умолчанию на конечных хостах, добавление статических маршрутов (`ip route`) через IP-адрес следующего перехода (`next-hop`), анализ таблиц маршрутизации (`show ip route`) и проверка связности.

## Топология

![Топология сети](images/топология.png)

* **Среда:** Cisco Packet Tracer
* **Сети:**
  * `192.168.1.0/24` — локальная сеть `PC1`
  * `192.168.12.0/24` — транзитный линк `R1 ↔ R2`
  * `192.168.13.0/24` — транзитный линк `R2 ↔ R3`
  * `192.168.3.0/24` — локальная сеть `PC2`

## Таблица адресации

| Устройство | Интерфейс | IPv4-адрес | Маска подсети | Шлюз по умолчанию |
| :--- | :--- | :--- | :--- | :--- |
| `PC1` | `FastEthernet0` | `192.168.1.1` | `255.255.255.0` | `192.168.1.254` |
| `R1` | `GigabitEthernet0/1` | `192.168.1.254` | `255.255.255.0` | — |
| `R1` | `GigabitEthernet0/0` | `192.168.12.1` | `255.255.255.0` | — |
| `R2` | `GigabitEthernet0/0` | `192.168.12.2` | `255.255.255.0` | — |
| `R2` | `GigabitEthernet0/1` | `192.168.13.2` | `255.255.255.0` | — |
| `R3` | `GigabitEthernet0/0` | `192.168.13.3` | `255.255.255.0` | — |
| `R3` | `GigabitEthernet0/1` | `192.168.3.254` | `255.255.255.0` | — |
| `PC2` | `FastEthernet0` | `192.168.3.1` | `255.255.255.0` | `192.168.3.254` |

## 1. Настройка интерфейсов маршрутизаторов

### Маршрутизатор `R1`

```cisco
R1(config)# interface GigabitEthernet0/1
R1(config-if)# ip address 192.168.1.254 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# interface GigabitEthernet0/0
R1(config-if)# ip address 192.168.12.1 255.255.255.0
R1(config-if)# no shutdown
```

![Настройка интерфейсов на R1](images/Настройка%20интерфейсов%20на%20маршрутизаторе%20R1.png)

### Маршрутизатор `R2`

```cisco
Router(config)# hostname R2
R2(config)# interface GigabitEthernet0/0
R2(config-if)# ip address 192.168.12.2 255.255.255.0
R2(config-if)# no shutdown
R2(config-if)# interface GigabitEthernet0/1
R2(config-if)# ip address 192.168.13.2 255.255.255.0
R2(config-if)# no shutdown
```

![Настройка интерфейсов на R2](images/настройка%20интерфейсов%20на%20маршрутизаторе%20R2.png)

### Маршрутизатор `R3`

```cisco
R3(config)# interface GigabitEthernet0/0
R3(config-if)# ip address 192.168.13.3 255.255.255.0
R3(config-if)# no shutdown
R3(config-if)# interface GigabitEthernet0/1
R3(config-if)# ip address 192.168.3.254 255.255.255.0
R3(config-if)# no shutdown
```

![Настройка интерфейсов на R3](images/настройка%20интерфейсов%20на%20маршрутизаторе%20R3.png)

## 2. Настройка конечных узлов (`PC1` и `PC2`)

### `PC1` (`192.168.1.1/24`, шлюз `192.168.1.254`)

![Настройка IP-адреса на PC1](images/настройка%20ip%20адреса%20на%20pc1.png)
![Настройка шлюза по умолчанию на PC1](images/настройка%20межсетевого%20шлюза%20для%20pc1.png)

### `PC2` (`192.168.3.1/24`, шлюз `192.168.3.254`)

![Настройка IP-адреса на PC2](images/настройка%20ip%20адреса%20на%20pc2.png)
![Настройка шлюза по умолчанию на PC2](images/настройка%20межсетевого%20шлюза%20для%20pc2.png)

## 3. Настройка статической маршрутизации и проверка таблиц

### Статический маршрут на `R1`

```cisco
R1(config)# ip route 192.168.3.0 255.255.255.0 192.168.12.2
```

Фрагмент таблицы маршрутизации `R1` (`show ip route`):

```cisco
C    192.168.1.0/24 is directly connected, GigabitEthernet0/1
L    192.168.1.254/32 is directly connected, GigabitEthernet0/1
S    192.168.3.0/24 [1/0] via 192.168.12.2
C    192.168.12.0/24 is directly connected, GigabitEthernet0/0
L    192.168.12.1/32 is directly connected, GigabitEthernet0/0
```

![Статический маршрут на R1](images/настройка%20статического%20пути%20для%20R1.png)

### Статические маршруты на `R2`

```cisco
R2(config)# ip route 192.168.1.0 255.255.255.0 192.168.12.1
R2(config)# ip route 192.168.3.0 255.255.255.0 192.168.13.3
```

Фрагмент таблицы маршрутизации `R2` (`show ip route`):

```cisco
S    192.168.1.0/24 [1/0] via 192.168.12.1
S    192.168.3.0/24 [1/0] via 192.168.13.3
C    192.168.12.0/24 is directly connected, GigabitEthernet0/0
L    192.168.12.2/32 is directly connected, GigabitEthernet0/0
C    192.168.13.0/24 is directly connected, GigabitEthernet0/1
L    192.168.13.2/32 is directly connected, GigabitEthernet0/1
```

![Статические маршруты на R2](images/настройка%20статического%20пути%20для%20R2.png)

### Статический маршрут на `R3`

```cisco
R3(config)# ip route 192.168.1.0 255.255.255.0 192.168.13.2
```

Фрагмент таблицы маршрутизации `R3` (`show ip route`):

```cisco
S    192.168.1.0/24 [1/0] via 192.168.13.2
C    192.168.3.0/24 is directly connected, GigabitEthernet0/1
L    192.168.3.254/32 is directly connected, GigabitEthernet0/1
C    192.168.13.0/24 is directly connected, GigabitEthernet0/0
L    192.168.13.3/32 is directly connected, GigabitEthernet0/0
```

![Статический маршрут на R3](images/настройка%20статического%20пути%20для%20R3.png)

## 4. Проверка связности

Проверка прохождения ICMP-пакетов с `PC1` (`192.168.1.1`) на `PC2` (`192.168.3.1`):

```text
C:\>ping 192.168.3.1

Pinging 192.168.3.1 with 32 bytes of data:

Reply from 192.168.3.1: bytes=32 time<1ms TTL=125
Reply from 192.168.3.1: bytes=32 time<1ms TTL=125
Reply from 192.168.3.1: bytes=32 time<1ms TTL=125
Reply from 192.168.3.1: bytes=32 time<1ms TTL=125

Ping statistics for 192.168.3.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
```

![Успешный ping от PC1 к PC2](images/успешная%20проверка%20соединения%20между%20конечными%20хостами%20%28ура%29.png)

* Первые три пакета при первом запуске `ping` ушли в таймаут (`Request timed out`), пока устройства по цепочке разрешали MAC-адреса по протоколу ARP.
* Повторный `ping` прошёл с `0% loss`, а значение `TTL=125` подтверждает прохождение трафика через все три маршрутизатора (`128 − 3 = 125`).

## Итог

В таблицах маршрутизации всех трёх маршрутизаторов добавлены статические маршруты с административной дистанцией `1` (`S ... [1/0]`), а успешный `ping` между `PC1` и `PC2` подтвердил полную двустороннюю связность между сетями `192.168.1.0/24` и `192.168.3.0/24`.