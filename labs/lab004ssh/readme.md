| S1 | VLAN 1 | 192.168.1.11 | 255.255.255.0 | 192.168.1.1 |
| PC-A | NIC | 192.168.1.3 | 255.255.255.0 | 192.168.1.1 |

## 1. Базовая настройка R1

На маршрутизаторе R1 были выполнены базовые настройки:

```cisco
enable
configure terminal
no ip domain-lookup
hostname R1
enable secret class
service password-encryption
banner motd #Unauthorized access prohibited!#
````

Интерфейсу G0/0/1 был назначен IP-адрес:
```
interface g0/0/1
ip address 192.168.1.1 255.255.255.0
no shutdown
exit
```

Настроена консоль:
```
line console 0
password cisco
login
exit
```

Настроены VTY-линии:
```
line vty 0 4
password cisco
login
exit
```

## 2. Настройка SSH на R1

Для работы SSH были заданы имя устройства и доменное имя:
```
hostname R1
ip domain-name lab.local
```

Включена версия SSH 2:
```
ip ssh version 2
```

Сгенерированы RSA-ключи длиной 2048 бит:
```
crypto key generate rsa
```

Для аутентификации создан локальный пользователь:
```
username admin secret Adm1nP@55
```

VTY-линии настроены на использование локальной базы пользователей и SSH:
```
line vty 0 4
login local
transport input ssh
exit

line vty 5 15
login local
transport input ssh
exit
```

Таким образом, при подключении по SSH пользователь проходит аутентификацию через локальную базу пользователей маршрутизатора.

## 3. Настройка S1

На коммутаторе S1 был настроен интерфейс управления VLAN 1:
```
interface vlan 1
ip address 192.168.1.11 255.255.255.0
```

Настроен шлюз по умолчанию:
```
ip default-gateway 192.168.1.1
```

Для работы SSH настроены имя устройства, доменное имя и версия SSH:
```
hostname S1
ip domain-name lab.local
ip ssh version 2
```

Создан локальный пользователь:
```
username admin secret Adm1nP@55
```

Сгенерированы RSA-ключи:
```
crypto key generate rsa
```

VTY-линии настроены следующим образом:
```
line vty 0 4
login local
transport input ssh
exit

line vty 5 15
login local
transport input ssh
exit
```

После настройки конфигурация была сохранена:
```
copy running-config startup-config
`

