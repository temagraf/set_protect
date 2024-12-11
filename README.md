# Домашнее задание к занятию  «`Защита сети`» - `Яковлев Артем`

**Домашнее задание выполните в Google Docs или в md-файле в вашем репозитории GitHub.** 

Для оформления вашего решения в GitHub можете воспользоваться [шаблоном](https://github.com/netology-code/sys-pattern-homework).

Название файла Google Docs должно содержать номер лекции и фамилию студента. Пример названия: «13.2. Защита хоста — Александр Александров».

Перед тем как выслать ссылку, убедитесь, что её содержимое не является приватным, то есть открыто на просмотр всем, у кого есть ссылка. Если необходимо прикрепить дополнительные ссылки, просто добавьте их в свой Google Docs.

Любые вопросы по решению задач задавайте в чате учебной группы.

------

## Подготовка к выполнению заданий

1. Подготовка защищаемой системы:

- установите **Suricata**,
- установите **Fail2Ban**.

```bash
#ubuntu
sudo apt update && sudo apt upgrade
sudo apt install software-properties-common
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update
sudo apt install -y suricata
sudo suricata-update
sudo suricata-update list-sources
#sudo suricata-update update-sources
systemctl status suricata.service
suricata -V

sudo nano /etc/suricata/suricata.yaml # EXTERNAL_NET: "any"
sudo systemctl restart suricata.service
systemctl status suricata.service

sudo apt install -y fail2ban
systemctl status fail2ban
sudo systemctl start fail2ban
fail2ban-client -V

sudo nano /etc/fail2ban/jail.conf # [sshd] enabled = true
sudo systemctl restart fail2ban.service
systemctl status fail2ban.service
```

2. Подготовка системы злоумышленника: установите **nmap** и **thc-hydra** либо скачайте и установите **Kali linux**.

Обе системы должны находится в одной подсети.

------

## Задание 1

Проведите разведку системы и определите, какие сетевые службы запущены на защищаемой системе:

**sudo nmap -sA < ip-адрес >**

**sudo nmap -sT < ip-адрес >**

**sudo nmap -sS < ip-адрес >**

**sudo nmap -sV < ip-адрес >**

По желанию можете поэкспериментировать с опциями: https://nmap.org/man/ru/man-briefoptions.html.


*В качестве ответа пришлите события, которые попали в логи Suricata и Fail2Ban, прокомментируйте результат.*



```bash
#ubuntu
sudo suricata -c /etc/suricata/suricata.yaml -i enp0s3
tail -f /var/log/suricata/fast.log
grep "192.168.56.103" /var/log/suricata/fast.log


tail /var/log/auth.log
grep "192.168.56.103" /var/log/auth.log
cat /var/log/fail2ban.log

#kali
sudo nmap -sA 192.168.56.104
sudo nmap -sS 192.168.56.104
sudo nmap -sT 192.168.56.104
sudo nmap -sV 192.168.56.104

hydra -L users.txt -P pass.txt 192.168.56.104 ssh
```
Сканирование nmap -sA (ACK) в логи suricata не попало
В логах fail2ban зафиксировано только сканирование nmap -sV (Version detection)

**Сканирование nmap -sS (suricata)**
![Сканирование nmap -sS](https://github.com/temagraf/set_protect/blob/main/img/13-3-1-nmap-sS.png "Сканирование nmap -sS")

**Сканирование nmap -sT (suricata)**
![Сканирование nmap -sT](https://github.com/temagraf/set_protect/blob/main/img/13-3-1-nmap-sT.png "Сканирование nmap -sT")

**Сканирование nmap -sV (suricata)**
![Сканирование nmap -sV](https://github.com/temagraf/set_protect/blob/main/img/13-3-1-nmap-sV.png "Сканирование nmap -sV")

**Сканирование nmap -sV (fail2ban)**
![Сканирование nmap -sV (fail2ban)](https://github.com/temagraf/set_protect/blob/main/img/13-3-2-nmap-sV-fail2ban-log.png "Сканирование nmap -sV (fail2ban)")

------

## Задание 2

Проведите атаку на подбор пароля для службы SSH:

**hydra -L users.txt -P pass.txt < ip-адрес > ssh**

1. Настройка **hydra**: 
 
 - создайте два файла: **users.txt** и **pass.txt**;
 - в каждой строчке первого файла должны быть имена пользователей, второго — пароли. В нашем случае это могут быть случайные строки, но ради эксперимента можете добавить имя и пароль существующего пользователя.

Дополнительная информация по **hydra**: https://kali.tools/?p=1847.

2. Включение защиты SSH для Fail2Ban:

-  открыть файл /etc/fail2ban/jail.conf,
-  найти секцию **ssh**,
-  установить **enabled**  в **true**.

Дополнительная информация по **Fail2Ban**:https://putty.org.ru/articles/fail2ban-ssh.html.

*В качестве ответа пришлите события, которые попали в логи Suricata и Fail2Ban, прокомментируйте результат.*


После включения защиты ssh для Fail2Ban падает скорость перебора паролей. По умолчанию после 5 не верных паролей IP адрес хоста блокируется на 10 минут.

**Подбор пароля по ssh (fail2ban - до включения защиты ssh)**
![Подбор пароля по ssh (fail2ban)](https://github.com/temagraf/set_protect/blob/main/img/13-3-2-hydra-fail2ban-off.png "Подбор пароля по ssh (fail2ban - до включения защиты ssh)")

**Подбор пароля по ssh (fail2ban - после включения защиты ssh)**
![Подбор пароля по ssh (fail2ban)](https://github.com/temagraf/set_protect/blob/main/img/13-3-2-hydra-fail2ban-on.png "Подбор пароля по ssh (fail2ban - полсе включения защиты ssh)")

**Подбор пароля по ssh (suricata)**
![Подбор пароля по ssh (suricata)](https://github.com/temagraf/set_protect/blob/main/img/13-3-2-hydra-suricata.png "Подбор пароля по ssh (suricata)")

---
# Исправления :

- В логе /var/log/auth.log видим историю подключений, в том числе неудачные (подбор паролей) - см. скриншот лога ниже;
- В логе /var/log/fail2ban.log можем увидить факт блокировки IP (Ban или already banned) - см. скриншот лога ниже;
- Hydra с настройками по умолчанию и с users.txt из 6-и пользователей и с pass.txt из 6-и паролей
**до включения fail2ban** завершила работу за 10 секунд без ошибок - см. скриншот работы hydra ниже;
**после включения fail2ban** завершила работу через 43 секунды и с ошибками по причине refused connection - блокировка выполнена 

**Скриншот лога auth.log**
![Скриншот лога auth.log](https://github.com/temagraf/set_protect/blob/main/img/13-3-2-auth.log.png "Скриншот лога auth.log")

**Скриншот лога fail2ban.log**
![Скриншот лога fail2ban.log](https://github.com/temagraf/set_protect/blob/main/img/13-3-2-fail2ban.log.png "Скриншот лога fail2ban.log")

**Скриншот работы hydra**
![Скриншот работы hydra](https://github.com/temagraf/set_protect/blob/main/img/13-3-2-Hydra.png "Скриншот работы hydra")

```bash
sudo fail2ban-client status # смотрим список контейнеров (jails)
sudo fail2ban-client status sshd # смотрим список заблокированных IP адресов по ssh
sudo fail2ban-client set sshd unbanip 192.168.56.103 # исключаем заблокированный IP
```

------
