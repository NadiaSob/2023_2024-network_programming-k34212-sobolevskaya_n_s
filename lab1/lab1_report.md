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

<img width="909" alt="image_2023-11-08_17-55-13" src="https://github.com/NadiaSob/2023_2024-network_programming-k34212-sobolevskaya_n_s/assets/43678322/f9126bc7-ab6f-4b83-8921-ca4722fd0c09">

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
<img width="634" alt="image_2023-11-08_18-10-21" src="https://github.com/NadiaSob/2023_2024-network_programming-k34212-sobolevskaya_n_s/assets/43678322/28a63b06-c460-4787-9077-a8110848f0c8">

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

<img width="626" alt="image_2023-11-08_18-15-41" src="https://github.com/NadiaSob/2023_2024-network_programming-k34212-sobolevskaya_n_s/assets/43678322/08fcc4de-414f-4869-9ca2-6fdec3b975ba">

Выводим ip адрес и подключаемся к роутеру через Winbox

<img width="226" alt="image_2023-11-08_18-20-28" src="https://github.com/NadiaSob/2023_2024-network_programming-k34212-sobolevskaya_n_s/assets/43678322/0797a68e-a1e0-4a91-9d32-5e0c56d708ef">

В Winbox подключаемся к адресу 192.168.0.106 и создаем подключение к виртуальной машине с адресом 84.201.161.145

![image](https://github.com/NadiaSob/2023_2024-network_programming-k34212-sobolevskaya_n_s/assets/43678322/ecf855d4-3e06-4b4a-8899-edea9f63d89b)

5. Проверка сетевой связанности

Сделаем пинг на роутер

<img width="423" alt="image_2023-11-08_19-00-51" src="https://github.com/NadiaSob/2023_2024-network_programming-k34212-sobolevskaya_n_s/assets/43678322/30bab935-ae42-4e89-8c2a-aeaad4fbdf40">

Сделаем пинг с роутера на виртуалку

<img width="338" alt="image_2023-11-08_18-41-48" src="https://github.com/NadiaSob/2023_2024-network_programming-k34212-sobolevskaya_n_s/assets/43678322/a9bacd55-1ea8-4d60-bfe8-6fe57f3c985d">

В результате получается схема связи следующего вида:

<img width="219" alt="image" src="https://github.com/NadiaSob/2023_2024-network_programming-k34212-sobolevskaya_n_s/assets/43678322/7d04b979-5567-455a-9fbe-0d68e95b6acf">

6. Вывод

В результате выполнения лабораторной работы была создана виртуальная машина, удалось поднять и настроить VPN сервер. Был установлен виртуальный роутер и настроен VPN тунель между сервером автоматизации и локальным CHR. Была проверена ip связанность.
