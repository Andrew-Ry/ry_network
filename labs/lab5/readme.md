# Технология VLAN и маршрутизация между VLAN

## Цель работы

Создать VLAN на двух коммутаторах, настроить транки IEEE 802.1Q и обеспечить маршрутизацию между VLAN методом Router-on-a-Stick на R1.

## Топология

```text
R1 G0/0/1 ─── F0/5 S1 F0/1 ─── F0/1 S2
                   │                  │
                 F0/6               F0/18
                   │                  │
                  PC-A               PC-B
```

- R1 — маршрутизатор, выполняющий маршрутизацию между VLAN.
- S1 и S2 — коммутаторы второго уровня.
- PC-A подключён к S1 F0/6 и находится в VLAN 20.
- PC-B подключён к S2 F0/18 и находится в VLAN 30.
- S1 F0/1 ↔ S2 F0/1 и S1 F0/5 ↔ R1 G0/0/1 — транковые соединения.

## Адресация и VLAN

| Устройство | Интерфейс | IP-адрес | Маска | Шлюз по умолчанию |
|---|---|---|---|---|
| R1 | G0/0/1.10 | 192.168.10.1 | 255.255.255.0 | — |
| R1 | G0/0/1.20 | 192.168.20.1 | 255.255.255.0 | — |
| R1 | G0/0/1.30 | 192.168.30.1 | 255.255.255.0 | — |
| R1 | G0/0/1.1000 | Не назначен | — | — |
| S1 | VLAN 10 | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 |
| S2 | VLAN 10 | 192.168.10.12 | 255.255.255.0 | 192.168.10.1 |
| PC-A | NIC | 192.168.20.3 | 255.255.255.0 | 192.168.20.1 |
| PC-B | NIC | 192.168.30.3 | 255.255.255.0 | 192.168.30.1 |

| VLAN | Имя | Назначение |
|---|---|---|
| 10 | Management | Управление коммутаторами |
| 20 | Sales | PC-A, S1 F0/6 |
| 30 | Operations | PC-B, S2 F0/18 |
| 999 | Parking_Lot | Неиспользуемые порты |
| 1000 | Native | Native VLAN транков |

В проверенном выводе S2 имя VLAN 30 записано как `operations`, а в одном из выводов S1 имя VLAN 999 — как `Parkink_Lot`. Регистр и эта опечатка не влияют на передачу трафика: VLAN определяется номером. Ниже приведены команды с именами `Operations` и `Parking_Lot` для единообразия.

## Порядок выполнения

1. Подключили устройства согласно топологии.
2. Задали имена устройств, пароли, предупреждающий баннер и отключили DNS-поиск.
3. Создали VLAN 10, 20, 30, 999 и 1000 на обоих коммутаторах.
4. Настроили виртуальные интерфейсы VLAN 10 для управления S1 и S2 и задали им шлюз по умолчанию.
5. Назначили порты компьютеров в VLAN 20 и 30 в режиме access.
6. Перевели неиспользуемые порты в VLAN 999 и административно выключили их.
7. Настроили транки с native VLAN 1000 и разрешёнными VLAN 10, 20, 30, 1000.
8. Включили G0/0/1 на R1 и настроили подинтерфейсы для маршрутизации.
9. Настроили адреса компьютеров, часы устройств и выполнили проверки связи.

Следующие блоки объединяют настройки в последовательности, пригодной для повторного ввода. Автоматические строки IOS и зашифрованные значения паролей из `show running-config` опущены.

## Настройка S1

```text
enable
configure terminal
hostname S1
no ip domain-lookup
enable secret class
service password-encryption
banner motd #Unauthorized access prohibited!#

line console 0
 password cisco
 login
 exit
line vty 0 15
 password cisco
 login
 exit

vlan 10
 name Management
 exit
vlan 20
 name Sales
 exit
vlan 30
 name Operations
 exit
vlan 999
 name Parking_Lot
 exit
vlan 1000
 name Native
 exit

interface vlan 10
 ip address 192.168.10.11 255.255.255.0
 no shutdown
 exit
ip default-gateway 192.168.10.1

interface fastEthernet 0/1
 switchport mode trunk
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 10,20,30,1000
 no shutdown
 exit

interface fastEthernet 0/5
 switchport mode trunk
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 10,20,30,1000
 no shutdown
 exit

interface fastEthernet 0/6
 switchport mode access
 switchport access vlan 20
 no shutdown
 exit

interface range fastEthernet 0/2-4, fastEthernet 0/7-24, gigabitEthernet 0/1-2
 switchport mode access
 switchport access vlan 999
 shutdown
 exit

clock timezone MSK 3
end
copy running-config startup-config
```

## Настройка S2

```text
enable
configure terminal
hostname S2
no ip domain-lookup
enable secret class
service password-encryption
banner motd #Unauthorized access prohibited!#

line console 0
 password cisco
 login
 exit
line vty 0 15
 password cisco
 login
 exit

vlan 10
 name Management
 exit
vlan 20
 name Sales
 exit
vlan 30
 name Operations
 exit
vlan 999
 name Parking_Lot
 exit
vlan 1000
 name Native
 exit

interface vlan 10
 ip address 192.168.10.12 255.255.255.0
 no shutdown
 exit
ip default-gateway 192.168.10.1

interface fastEthernet 0/1
 switchport mode trunk
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 10,20,30,1000
 no shutdown
 exit

interface fastEthernet 0/18
 switchport mode access
 switchport access vlan 30
 no shutdown
 exit

interface range fastEthernet 0/2-17, fastEthernet 0/19-24, gigabitEthernet 0/1-2
 switchport mode access
 switchport access vlan 999
 shutdown
 exit

clock timezone MSK 3
end
copy running-config startup-config
```

## Настройка R1

```text
enable
configure terminal
hostname R1
no ip domain-lookup
enable secret class
service password-encryption
banner motd #Unauthorized access prohibited!#

line console 0
 password cisco
 login
 exit
line vty 0 4
 password cisco
 login
 exit

interface gigabitEthernet 0/0/1
 description Trunk to S1 F0/5
 no ip address
 no shutdown
 exit

interface gigabitEthernet 0/0/1.10
 description Management VLAN 10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit

interface gigabitEthernet 0/0/1.20
 description Sales VLAN 20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 exit

interface gigabitEthernet 0/0/1.30
 description Operations VLAN 30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 exit

interface gigabitEthernet 0/0/1.1000
 description Native VLAN 1000
 encapsulation dot1Q 1000 native
 no ip address
 exit

clock timezone MSK 3
end
copy running-config startup-config
```

Физическому интерфейсу G0/0/1 IP-адрес не назначается. Адреса шлюзов назначены его подинтерфейсам. Для native VLAN 1000 создан отдельный подинтерфейс без IP-адреса. Подинтерфейс для VLAN 999 не требуется: неиспользуемые порты выключены, и маршрутизация для них не предусмотрена.

Статические маршруты и маршрут по умолчанию на R1 для этой топологии не нужны: сети VLAN 10, 20 и 30 подключены непосредственно к маршрутизатору.

## Настройка часов и сохранение

На устройствах настроили часовой пояс MSK, UTC+3. Время задаётся из привилегированного режима в 24-часовом формате:

```text
clock set 15:00:00 25 Oct 2026
show clock
```

При повторении работы необходимо подставить фактические дату и время. В ходе проверки получен вывод:

```text
15:0:12.857 MSK Sun Oct 25 2026
```

Настройка часов на остальных устройствах подтверждена пользователем.

Команды конфигурации сразу изменяют `running-config`. Для сохранения после перезагрузки используется:

```text
copy running-config startup-config
```

На запрос имени файла достаточно нажать Enter. Часовой пояс сохраняется в конфигурации, а значение, установленное через `clock set`, её частью не является. Проект симулятора также необходимо сохранять отдельно.

## Проверка коммутаторов

Использовали команды:

```text
show running-config
show vlan brief
show interfaces trunk
show ip interface brief
show interfaces fastEthernet 0/5 switchport
```

| Проверка | Полученный результат |
|---|---|
| VLAN на S1 и S2 | Присутствуют VLAN 10, 20, 30, 999 и 1000 |
| S1 F0/6 | Access VLAN 20, up/up |
| S2 F0/18 | Access VLAN 30, up/up |
| S1 F0/1 и S2 F0/1 | Работающие транки 802.1Q, native VLAN 1000 |
| Разрешённые VLAN межкоммутаторного транка | 10, 20, 30, 1000; находятся в состоянии пересылки |
| S1 F0/5 | Режим trunk, native VLAN 1000, разрешены 10,20,30,1000; после включения соединения — up/up |
| S1 Vlan10 | 192.168.10.11, up/up |
| S2 Vlan10 | 192.168.10.12, up/up |
| Неиспользуемые порты | VLAN 999, administratively down |

`show vlan brief` показывает созданные VLAN и их access-порты. `show ip interface brief` показывает интерфейсы, а не все VLAN. Поэтому на коммутаторах присутствует SVI VLAN 10, но отсутствуют SVI остальных VLAN: шлюзы пользовательских сетей находятся на R1.

Значение `unassigned` на физических портах коммутаторов является нормальным: IP-адрес управления назначен виртуальному интерфейсу VLAN 10. Поле `Access Mode VLAN: 1` у порта в режиме trunk не означает, что транк передаёт только VLAN 1.

## Проверка R1

Команда `show ip interface brief` подтвердила состояние `up/up` для G0/0/1 и подинтерфейсов .10, .20, .30, .1000.

Команда `show ip route` показала следующие записи:

```text
C 192.168.10.0/24 is directly connected, GigabitEthernet0/0/1.10
L 192.168.10.1/32 is directly connected, GigabitEthernet0/0/1.10
C 192.168.20.0/24 is directly connected, GigabitEthernet0/0/1.20
L 192.168.20.1/32 is directly connected, GigabitEthernet0/0/1.20
C 192.168.30.0/24 is directly connected, GigabitEthernet0/0/1.30
L 192.168.30.1/32 is directly connected, GigabitEthernet0/0/1.30
```

- `C` — непосредственно подключённая сеть.
- `L` — собственный IP-адрес интерфейса маршрутизатора, маршрут /32.
- Сообщение `Gateway of last resort is not set` допустимо: выход в другие сети в этой работе не настраивался.

## Проверка связи

С PC-A выполнены проверки:

| Команда | Что проверяет | Результат |
|---|---|---|
| `ping 192.168.20.1` | Доступность своего шлюза | Успешно |
| `ping 192.168.30.3` | Связь с PC-B через R1 | Успешно |
| `ping 192.168.10.12` | Доступность управления S2 из VLAN 20 | Успешно |

Успешное выполнение всех трёх ping подтверждено пользователем; подробная статистика пакетов в переписке не предоставлялась.

С PC-B выполнена трассировка к PC-A. Получен вывод:

```text
C:\>tracert 192.168.20.3

Tracing route to 192.168.20.3 over a maximum of 30 hops:

  1   0 ms      0 ms      4 ms      192.168.30.1
  2   1 ms      0 ms      1 ms      192.168.20.3

Trace complete.
```

Трассировка подтверждает путь PC-B → R1 → PC-A. Коммутаторы второго уровня не отображаются как IP-переходы.

## Ответы на вопросы задания

### Что произойдёт при отключении G0/0/1 на R1

При отключении физического интерфейса G0/0/1 все его подинтерфейсы также перестанут работать. Шлюзы VLAN 10, 20 и 30 станут недоступны, а маршрутизация между этими VLAN прекратится.

Связь внутри одной VLAN через работающие коммутаторы и межкоммутаторный транк сохранится. Например, S1 и S2 смогут обмениваться трафиком через свои адреса управления в VLAN 10. Между PC-A и PC-B связи не будет, поскольку они находятся в разных VLAN.

Это объяснение ожидаемого поведения; отдельный эксперимент с отключением интерфейса в переписке не зафиксирован.

### Какие промежуточные IP адреса показывает tracert с PC-B до PC-A

Промежуточный IP-адрес — **192.168.30.1**, адрес подинтерфейса G0/0/1.30 маршрутизатора R1 и шлюз PC-B.

Второй адрес, **192.168.20.3**, принадлежит конечному узлу PC-A. Поэтому в трассировке два перехода, из которых один — промежуточный маршрутизатор.

## Разобранные ошибки и исправления

- На S2 маска `255.255.255.0` сначала была введена как шлюз. Команда `ip default-gateway 192.168.10.1` заменила ошибочное значение. Маска задаётся вместе с IP-адресом командой `ip address` в настройках интерфейса.
- На S1 ошибочно настраивался F0/18. Его вернули в VLAN 999 и выключили; для PC-A настроили F0/6 в VLAN 20.
- На S2 отсутствовали VLAN 20 и 1000, хотя они были разрешены на транке. Создали их явно: список разрешённых VLAN не создаёт сами VLAN.
- SVI Vlan10 первоначально имел состояние up/down. После создания необходимых VLAN и настройки активного транка состояние стало up/up.
- S1 F0/5 первоначально был down/down. После включения соединения с R1 стал up/up.
- Часы показывали 1993 год. Установили дату, время и московский часовой пояс.

## Вывод

Созданы отдельные VLAN управления, Sales и Operations. Настроены транки IEEE 802.1Q, native VLAN 1000 и изоляция неиспользуемых портов в VLAN 999. R1 выполняет маршрутизацию между сетями 192.168.10.0/24, 192.168.20.0/24 и 192.168.30.0/24 через подинтерфейсы одного физического порта. Успешные ping и трассировка подтвердили связь между VLAN.

## Исходные материалы

- Лекция «VLAN и маршрутизация между VLAN»: `NG_OTUS_13_VLAN-35585-eedb3b (1).pdf`.
- Задание лабораторной работы: `Lab___Implement_Inter_VLAN_Routing-1801-a988ef (1).docx`.
- Выводы команд и результаты проверок, предоставленные в ходе выполнения работы.

Адресация и номера VLAN взяты из задания. Пример VLAN 99 из лекции в конфигурации этой работы не использовался.
