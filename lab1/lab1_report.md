University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2023/2024

Group: K34212

Author: Sobolevskaya Nadezhda Sergeevna

Lab: Lab1

Date of create: 05.11.2023

Date of finished: 08.11.2023

# Отчет по лабораторной работе №1: "Установка CHR и Ansible, настройка VPN"

## Цель работы

Целью данной работы является развертывание виртуальной машины на базе платформы Microsoft Azure с установленной системой контроля конфигураций Ansible и установка CHR в VirtualBox

## Ход работы

1. Создаем виртуальную машину и подключаемся к ней

![Alt text](image.png)

```
PS C:\Users\sobo_> ssh admin@84.201.161.145
```
2. Установим python3 и Ansible

```
sudo apt install python3-pip
ls -la /usr/bin/python3.6
sudo pip3 install ansible
ansible --version
```
![Alt text](image-1.png)

3. Настроим VPN сервер

Устанавливаем ppp pptpd

```
apt-get install ppp pptpd
echo "localip 192.168.0.1
remoteip 192.168.0.2-200" >> /etc/pptpd.conf
echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
echo "mtu 1400
mru 1400
auth
require-mppe" >> /etc/ppp/pptpd-options
iptables -A INPUT -p gre -j ACCEPT
iptables -A INPUT -m tcp -p tcp --dport 1723 -j ACCEPT
iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
echo 'chr1  pptpd  password1  "*"' >> /etc/ppp/chap-secrets
service pptpd restart

```

В файл `/etc/pptpd.conf` добавляем пул ip адресов:

```
localip 192.168.0.1 - локальный шлюз для клиентов VPN
remoteip 192.168.0.2-100 - для раздачи клиентам VPN
```

В файле `/etc/sysctl.conf` раскомментируем строку

```
net.ipv4.ip_forward=1` 
```

Добавляем правила в iptables, выполним команды:
```
iptables -A INPUT -p gre -j ACCEPT
iptables -A INPUT -m tcp -p tcp --dport 1723 -j ACCEPT
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

Добавляем пользователя для подключения к VPN серверу в файле `/etc/ppp/chap-secrets`
```
user1	pptpd	password1	"*"
```

4. Установим CHR на VirtualBox

Образ для CHR был взят с сайта mikrotik.com. Была установлена версия 6.49.10 Long-term.
Загруженный файл открываем в VirtualBox и запускаем машину.

![Alt text](image_2023-11-08_18-15-41.png)

Выводим ip адрес и подключаемся к роутеру через Winbox

![Alt text](image-2.png)

В Winbox подключаемся к адресу 192.168.0.106 и создаем подключение к виртуальной машине с адресом 84.201.161.145

![Alt text](image-3.png)

5. Проверка сетевой связанности

Сделаем пинг на роутер

![Alt text](image-5.png)

Сделаем пинг с роутера на виртуалку

![Alt text](image_2023-11-08_18-41-48.png)

В рехультате получается схема связи следующего вида:

![Alt text](image-6.png)

6. Вывод

В результате выполнения лабораторной работы была создана виртуальная машина, удалось поднять и настроить VPN сервер. Был установлен виртуальный роутер и настроен VPN тунель между сервером автоматизации и локальным CHR. Была проверена ip связанность.
