# Настройка VLAN и маршрутизация между VLAN (Inter-VLAN Routing)

Лабораторная работа по сегментации локальной сети на уровне доступа с использованием **VLAN** на коммутаторе `SW1` (`VLAN 10`, `VLAN 20`, `VLAN 30`), разбиению адресного пространства `10.0.0.0/24` на подсети `/26`, организации маршрутизации между VLAN через отдельные физические интерфейсы маршрутизатора `R1` и проверке изоляции широковещательных доменов.

## Топология и задание

![Топология сети и задание](images/топология.png)

* **Среда:** Cisco Packet Tracer
* **Базовая сеть:** `10.0.0.0/24` (подсети `/26`, маска `255.255.255.192`)

**Задачи:**
1. Настроить IP-адреса и маски подсети на `PC1`–`PC6`; в качестве шлюза по умолчанию задать **последний доступный** IP-адрес соответствующей подсети.
2. Подключить `R1` к `SW1` тремя отдельными линками и настроить на `R1` по одному интерфейсу-шлюзу для каждого VLAN.
3. Назначить порты `SW1` (включая аплинки к `R1`) в соответствующие VLAN и задать имена (`Engineering`, `HR`, `Sales`).
4. Проверить связность между ПК из разных VLAN и выполнить широковещательный `ping` (на broadcast-адрес подсети), чтобы убедиться в изоляции широковещательных доменов.

## Таблица VLAN и подсетей

| VLAN ID | Имя VLAN | Подсеть | Маска подсети | Шлюз (`R1`) | Broadcast-адрес | Порты `SW1` |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- |
| `10` | `Engineering` | `10.0.0.0/26` | `255.255.255.192` | `10.0.0.62` | `10.0.0.63` | `Gig0/1`, `Fa3/1`, `Fa4/1` |
| `20` | `HR` | `10.0.0.64/26` | `255.255.255.192` | `10.0.0.126` | `10.0.0.127` | `Gig1/1`, `Fa5/1`, `Fa6/1` |
| `30` | `Sales` | `10.0.0.128/26` | `255.255.255.192` | `10.0.0.190` | `10.0.0.191` | `Gig2/1`, `Fa7/1`, `Fa8/1` |

## Таблица адресации устройств

| Устройство | Интерфейс / Порт `SW1` | VLAN | IPv4-адрес | Маска подсети | Шлюз по умолчанию |
| :--- | :--- | :---: | :--- | :--- | :--- |
| `R1` | `GigabitEthernet0/0` (`SW1 Gig0/1`) | `10` | `10.0.0.62` | `255.255.255.192` | — |
| `R1` | `GigabitEthernet0/1` (`SW1 Gig1/1`) | `20` | `10.0.0.126` | `255.255.255.192` | — |
| `R1` | `GigabitEthernet0/2` (`SW1 Gig2/1`) | `30` | `10.0.0.190` | `255.255.255.192` | — |
| `PC1` | `FastEthernet0` (`SW1 Fa3/1`) | `10` | `10.0.0.1` | `255.255.255.192` | `10.0.0.62` |
| `PC2` | `FastEthernet0` (`SW1 Fa4/1`) | `10` | `10.0.0.2` | `255.255.255.192` | `10.0.0.62` |
| `PC3` | `FastEthernet0` (`SW1 Fa5/1`) | `20` | `10.0.0.65` | `255.255.255.192` | `10.0.0.126` |
| `PC4` | `FastEthernet0` (`SW1 Fa6/1`) | `20` | `10.0.0.66` | `255.255.255.192` | `10.0.0.126` |
| `PC5` | `FastEthernet0` (`SW1 Fa7/1`) | `30` | `10.0.0.129` | `255.255.255.192` | `10.0.0.190` |
| `PC6` | `FastEthernet0` (`SW1 Fa8/1`) | `30` | `10.0.0.130` | `255.255.255.192` | `10.0.0.190` |

---

## 1. Настройка конечных узлов (`PC1`–`PC6`)

### VLAN 10 (`Engineering` — `10.0.0.0/26`, шлюз `10.0.0.62`)

![Настройка IPv4 на PC1](images/ip%20pc1.png)
![Настройка IPv4 на PC2](images/ip%20pc2.png)
![Шлюз по умолчанию для PC1 и PC2](images/pc1%20-%20pc2%20default%20gateway.png)

### VLAN 20 (`HR` — `10.0.0.64/26`, шлюз `10.0.0.126`)

![Настройка IPv4 на PC3](images/ip%20pc3.png)
![Настройка IPv4 на PC4](images/ip%20pc4.png)
![Шлюз по умолчанию для PC3 и PC4](images/pc3%20-%20pc4%20default%20gateway.png)

### VLAN 30 (`Sales` — `10.0.0.128/26`, шлюз `10.0.0.190`)

![Настройка IPv4 на PC5](images/ip%20pc5.png)
![Настройка IPv4 на PC6](images/ip%20pc6.png)
![Шлюз по умолчанию для PC5 и PC6](images/pc5%20-%20pc6%20default%20gateway.png)

---

## 2. Настройка интерфейсов маршрутизатора `R1`

На маршрутизаторе `R1` назначены IP-адреса шлюзов по умолчанию для каждой из трёх подсетей, после чего все три интерфейса одновременно включены через `interface range`:

```cisco
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip address 10.0.0.62 255.255.255.192
R1(config-if)# interface GigabitEthernet0/1
R1(config-if)# ip address 10.0.0.126 255.255.255.192
R1(config-if)# interface GigabitEthernet0/2
R1(config-if)# ip address 10.0.0.190 255.255.255.192
R1(config-if)# interface range GigabitEthernet0/0 - 2
R1(config-if-range)# no shutdown
```

Проверка состояния интерфейсов (`show ip interface brief`):

```cisco
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     10.0.0.62       YES manual up                    up
GigabitEthernet0/1     10.0.0.126      YES manual up                    up
GigabitEthernet0/2     10.0.0.190      YES manual up                    up
```

![Настройка и включение интерфейсов на R1](images/настройка%20интерфейсов%20на%20R1.png)

---

## 3. Настройка VLAN и портов доступа на коммутаторе `SW1`

### Назначение аплинков к `R1` и автоматическое создание VLAN

При назначении портов `Gig0/1`, `Gig1/1` и `Gig2/1` в несуществующие VLAN 10, 20 и 30 Cisco IOS автоматически создаёт соответствующие VLAN:

```cisco
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# interface GigabitEthernet1/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# interface GigabitEthernet2/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 30
```

![Настройка VLAN access-портов для аплинков на SW1](images/настройка%20vlan%20access%20портов%20на%20коммутаторе.png)

### Именование VLAN

```cisco
SW1(config)# vlan 10
SW1(config-vlan)# name Engineering
SW1(config-vlan)# vlan 20
SW1(config-vlan)# name HR
SW1(config-vlan)# vlan 30
SW1(config-vlan)# name Sales
```

![Настройка имён VLAN на SW1](images/настройка%20алиасов%20для%20vlan.png)

### Назначение портов конечных узлов (`PC1`–`PC6`)

```cisco
SW1(config)# interface range FastEthernet3/1, FastEthernet4/1
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 10
SW1(config-if-range)# interface range FastEthernet5/1, FastEthernet6/1
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 20
SW1(config-if-range)# interface range FastEthernet7/1, FastEthernet8/1
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 30
```

Итоговая таблица VLAN (`show vlan brief`):

```cisco
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa9/1
10   Engineering                      active    Gig0/1, Fa3/1, Fa4/1
20   HR                               active    Gig1/1, Fa5/1, Fa6/1
30   Sales                            active    Gig2/1, Fa7/1, Fa8/1
```

![Настройка access-портов ПК и проверка show vlan brief](images/настройка%20access%20портов%20на%20коммутаторе%20%28pc%29.png)

---

## 4. Проверка связности и широковещательных доменов

### Межсетевая связность (Inter-VLAN Ping)

Проверка `ping` с `PC1` (`10.0.0.1`, `VLAN 10`) на узлы `VLAN 30` (`PC6` `10.0.0.130` и `PC5` `10.0.0.129`). После первого таймаута на ARP-разрешение все пакеты проходят с `0% loss`, а `TTL=127` подтверждает маршрутизацию через `R1`:

![Проверка ping PC1 — PC6 (10.0.0.130)](images/проверка%20доступности%20хостов%20сети.png)
![Проверка ping до PC5 (10.0.0.129)](images/проверка%20доступности%20хостов%20сети%20%282%29.png)

### Проверка изоляции широковещательных доменов (Broadcast Ping)

При отправке ICMP-запросов на широковещательный адрес своей подсети ответы приходят **только от устройств внутри того же VLAN** (соседний ПК с `TTL=128` и шлюз `R1` с `TTL=255`). Маршрутизатор не пересылает широковещательный трафик в другие VLAN:

* **VLAN 10 (`PC2` → `10.0.0.63`):** ответили только `PC1` (`10.0.0.1`) и `R1 G0/0` (`10.0.0.62`):

  ![Проверка широковещательного домена VLAN 10](images/проверка%20широковещательного%20домена%20vlan%2010.png)

* **VLAN 20 (`PC3` → `10.0.0.127`):** ответили только `PC4` (`10.0.0.66`) и `R1 G0/1` (`10.0.0.126`):

  ![Проверка широковещательного домена VLAN 20](images/проверка%20широковещательного%20домена%20vlan%2020.png)

* **VLAN 30 (`PC5` → `10.0.0.191`):** ответили только `PC6` (`10.0.0.130`) и `R1 G0/2` (`10.0.0.190`):

  ![Проверка широковещательного домена VLAN 30](images/проверка%20широковещательного%20домена%20vlan%2030.png)