# Linux-server protection utilities

## Fail2Ban
Програма вміє боротися з різними атаками на всі популярні NIX-сервіси, такі як Apache, Nginx, ProFTPD, vsftpd, Exim, Postfix, named, і т.д.
Але насамперед Fail2ban відомий завдяки готовності «з коробки» до захисту SSH-сервера від атак типу «bruteforce», тобто захисту SSH від перебору паролів.

Інсталювання

```apt-get install fail2ban```

У програми два файли конфігурації: 
1. /etc/fail2ban/fail2ban.conf — відповідає за налаштування запуску процеса Fail2ban.
2. /etc/fail2ban/jail.conf — містить налаштування захисту конкретних сервисів, в тому числі sshd.

Не варто редагувати основний файл налаштувань jail.conf, для цього передбачені файли з розширенням *.local, які автоматично підключаються та мають найвищий пріоритет.
```/etc/fail2ban/jail.local```

```
[DEFAULT]
 ## Ignored IP adresses (newer banned)
 ignoreip = 212.101.101.101
 
 [ssh]
 ## Find in period:
 findtime    = 3600
 # Quantity of unsuccessfull logins:
 maxretry    = 6
 # 24 hours bantime
 bantime     = 86400
 ```
  
 Після налаштувань перезапускаємо сервіс fail2ban
 ```service fail2ban restart```
 
 Перевірка логування fail2ban
 ```tail /var/log/fail2ban.log```
 
 Перегляд статуса забанених Ip
 ```fail2ban-client status sshd```
 
 
  ## Port Knocking
 
 Інсталювання
 ```sudo apt install knockd -y```
 
 Редагуємо файл /etc/knockd.conf
  ```
 [options]
         UseSyslog
 
 [openSSH]
         sequence    = 9010,9080
         seq_timeout = 5
         command     = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
         tcpflags    = syn
 
 [closeSSH]
         sequence    = 9080,9010
         seq_timeout = 5
         command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
         tcpflags    = syn
 ```
 
 Редагуємо файл /etc/default/knockd
```
# control if we start knockd at init or not
# 1 = start
# anything else = don't start
# PLEASE EDIT /etc/knockd.conf BEFORE ENABLING
START_KNOCKD=1

# command line options
KNOCKD_OPTS="-i enp2s0"
```

Додамо правило в кінець! списку на блокування порту 22 SSH (-A - в кінець списку)

```sudo iptables -A INPUT -p tcp --dport 22 -j REJECT```

Перевірка і запуск служби port knocking
```
$ sudo systemctl start knockd
$ sudo systemctl enable knockd
```

Якщо виникає помилка при enable knockd:
add the following to the end of /lib/systemd/system/knockd.service
```
[Install]
WantedBy=multi-user.target
Alias=knockd.service
```

Зєднання з сервером
```
$ knock -v 95.216.21.107 9010 9080
$ ssh root@95.216.21.107
```

###Довідка:

Вивести список IPtables 
```sudo iptables -L --line-numbers```
Видалити вручну записи з IP-tables
```iptables -D INPUT 2```
