## Применить файловые лимиты в рамках текущей сессии с помощью утилиты prlimit:

```
prlimit --pid $$ --nofile=100000:100000
```

## Количество текущих открытых файлов

```
 $ cat /proc/sys/fs/file-nr
7744    521      92233720
```

- 7744 — суммарное количество открытых файлов
- 521– число открытых файлов, но которые сейчас не используются
- 92233720– максимальное количество файлов, которое разрешено открывать

## Увеличить лимит открытых файловых дескрипторов для процесса:

```
ulimit -n 8192
sudo sh -c "ulimit -n 8192"
```

## Как увеличить лимит открытых файлов в Linux

### Поиск лимита открытых файлов в Linux

Значение хранится в:

```bash
# cat /proc/sys/fs/file-max
818354
```

Число, которое вы увидите, показывает количество файлов, которые пользователь может открыть за одну сессию входа. Результат может отличаться в зависимости от вашей системы.

Например, на одном из моих серверов CentOS лимит был установлен в 818354, в то время как на Ubuntu-сервере, который я использую дома, значение по умолчанию было установлено в 176772.

Если вы хотите увидеть жесткие и мягкие лимиты, вы можете использовать следующие команды:

### Проверка жесткого лимита в Linux

```bash
# ulimit -Hn
4096
```

### Проверка мягких лимитов в Linux

```bash
# ulimit -Sn
1024
```

Чтобы увидеть жесткие и мягкие значения для разных пользователей, вы можете просто переключиться на пользователя с помощью "su" к пользователю, лимиты которого вы хотите проверить.

Например:

```bash
# su marin
$ ulimit -Sn
1024
$ ulimit -Hn
4096
```

### Как проверить лимиты файловых дескрипторов на уровне системы в Linux

Если вы управляете сервером, некоторые ваши приложения могут требовать более высоких лимитов для открытых файловых дескрипторов. Хорошим примером таких приложений являются службы MySQL/MariaDB или веб-сервер Apache.

Вы можете увеличить лимит открытых файлов в Linux, изменив директиву ядра fs.file-max. Для этого вы можете использовать утилиту sysctl.

Sysctl используется для конфигурации параметров ядра во время выполнения.

Например, чтобы увеличить лимит открытых файлов до 500000, вы можете использовать следующую команду от имени root:

```bash
# sudo sysctl -w fs.file-max=500000
```

Вы можете проверить текущее значение открытых файлов с помощью следующей команды:

```bash
$ cat /proc/sys/fs/file-max
```

С помощью этой команды изменения останутся активными только до следующей перезагрузки. Если вы хотите, чтобы они применялись постоянно, вам придется отредактировать следующий файл:

```bash
# sudo vim /etc/sysctl.conf
```

Добавьте следующую строку:

```bash
fs.file-max=500000
```

Конечно, вы можете изменить число по своему усмотрению. Для повторной проверки изменений используйте:

```bash
# cat /proc/sys/fs/file-max
```

Пользователи должны выйти и войти снова, чтобы изменения вступили в силу. Если вы хотите применить лимит немедленно, вы можете использовать следующую команду:

```bash
# sudo sysctl -p
```

### Установка лимитов открытых файлов на уровне пользователя в Linux

Приведенные выше примеры показывают, как установить глобальные лимиты, но вы можете захотеть устанавливать лимиты на уровне каждого пользователя. Для этого, как пользователь root, вам нужно отредактировать следующий файл:

```bash
# sudo vim /etc/security/limits.conf
```

Если вы являетесь администратором Linux, я рекомендую вам хорошо ознакомиться с этим файлом и тем, что вы можете с ним делать. Прочитайте все комментарии в нем, так как он предоставляет большую гибкость в управлении ресурсами системы, ограничивая пользователей/группы на различных уровнях.

Строки, которые вы должны добавить, имеют следующие параметры:

```bash
<domain>        <type>  <item>  <value>
```

Вот пример установки мягкого и жесткого лимитов для пользователя marin:

```bash
## Пример жесткого лимита для максимального числа открытых файлов
marin        hard nofile 4096
## Пример мягкого лимита для максимального числа открытых файлов
marin        soft nofile 1024
```

## Как в linux посмотреть у каких приложений больше всего открытых файлов?

```
lsof | awk '{print $1}' | sort | uniq -c | sort -nr | head
```

Эта команда:

- `lsof` выводит список всех открытых файлов
- `awk '{print $1}'` выводит только имена процессов, владеющих файлами  
- `sort` сортирует
- `uniq -c` подсчитывает количество для каждого процесса
- Второй `sort -nr` сортирует по убыванию
- `head` выводит топ 10 процессов с наибольшим количеством файлов

## resolv.conf 

/etc/NetworkManager/conf.d/DontTouchDNSResolution.conf

```
[main]
dns=none
systemd-resolved=false
```

Чтоб заставить NetworkManager перестать управлять резолвингом, создадим файл /etc/NetworkManager/conf.d/dns.conf со следующим содержанием:

```
[main]
dns=none
```

Подробнее о конфигурации NetworkManager можно прочитать здесь. Убедимся, что опция dns нигде больше не задана

```
grep -r dns= /etc/NetworkManager
```

Если задана, то закомментируйте ее.

Применим конфигурацию

```
systemctl reload NetworkManager.service
```

Теперь настроим локальный кеширующий dnsmasq, кстати, я уже писал о нем.

Пропишем конфиг в /etc/dnsmasq.d/local.conf

```
server=1.1.1.1
server=2606:4700:4700::1111
listen-address=127.0.0.1
cache-size=10000
no-negcache
no-resolv
bind-interfaces
```

Я тут использую Cloudflare public DNS, но вы можете настроить любой на свой вкус.

Пропишем в /etc/resolv.conf наш новый резолвер:

```
# generated manually
nameserver 127.0.0.1
```

Включаем в загрузку и стартуем dnsmasq:

```
systemctl enable dnsmasq.service
systemctl start dnsmasq.service
```

## Изменить размер свопа `swap`

```bash
sudo swapoff -a
sudo dd if=/dev/zero of=/swapfile bs=1G count=8
sudo mkswap /swapfile
sudo swapon /swapfile
grep SwapTotal /proc/meminfo
```

## Генерация ssh ключей git (Gitlab)

Создайте пару ключей SSH
Если нет существующей пары ключей SSH, сгенерируйте новую.

Например, для ED25519:
```
ssh-keygen -t ed25519 -C "<comment>"
```

Для 2048-битного RSA:
```
ssh-keygen -t rsa -b 2048 -C "<comment>"
```

Нажмите Ввод. Отображается результат, подобный следующему:

```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/user/.ssh/id_rsa):
```

Добавьте SSH-ключ в учетную запись GitLab
Чтобы использовать SSH с GitLab, скопируйте открытый ключ в учетную запись GitLab.

```
cat /home/user/.ssh/id_rsa.pub
```

## Просмотр размера файлов

### Вариант 1

Существует также ncurses интерфейс для du с соответствующим названием ncdu, который вы можете установить:

```
sudo apt install ncdu
```

Это графически отобразит использование вашего диска:

```console
$ ncdu
--- /root ----------------------------------------------------------------------
    8.0KiB [##########] /.ssh                                                   
    4.0KiB [#####     ] /.cache
    4.0KiB [#####     ]  .bashrc
    4.0KiB [#####     ]  .profile
    4.0KiB [#####     ]  .bash_history
```

## Мониторинг сети

### Вариант 1

Если кажется, что ваше сетевое подключение перегружено и вы не уверены, какое приложение является виновником, программа под названием nethogs - хороший выбор для выяснения этого.

В Ubuntu вы можете установить nethogs с помощью следующей команды:

```
sudo apt install nethogs
```

После этого будет доступна команда nethogs:

```console
$ nethogs
NetHogs version 0.8.0

  PID USER     PROGRAM                      DEV        SENT      RECEIVED       
3379  root     /usr/sbin/sshd               eth0       0.485       0.182 KB/sec
820   root     sshd: root@pts/0             eth0       0.427       0.052 KB/sec
?     root     unknown TCP                             0.000       0.000 KB/sec

  TOTAL                                                0.912       0.233 KB/sec
```

nethogs связывает каждое приложение с его сетевым трафиком.

Есть только несколько команд, которые вы можете использовать для управления nethogs:

- M: Измените отображение между "kb / s“, “kb”, “b” и “mb".
- R: Сортировка по полученному трафику.
- S: Сортировка по отправленному трафику.
- Q: Выйти

### Вариант 2

iptraf-ng это еще один способ мониторинга сетевого трафика. Он предоставляет ряд различных интерактивных интерфейсов мониторинга.

Примечание: для IPTraf требуется размер экрана не менее 80 столбцов по 24 строки.

В Ubuntu вы можете установить iptraf-ng с помощью следующей команды:

```
sudo apt install iptraf-ng
```

iptraf-ng должен запускаться с правами root, поэтому вам следует предварить его sudo:

```
sudo iptraf-ng
```

### Вариант 3

Команда netstat - еще один универсальный инструмент для сбора сетевой информации.

netstat установлен по умолчанию в большинстве современных систем, но вы можете установить его самостоятельно, загрузив из репозиториев пакетов вашего сервера по умолчанию. В большинстве систем Linux, включая Ubuntu, пакет, содержащий netstat является net-tools:

```
sudo apt install net-tools
```

По умолчанию команда netstat сама выводит список открытых сокетов:

```console
$ netstat
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 192.241.187.204:ssh     ip223.hichina.com:50324 ESTABLISHED
tcp        0      0 192.241.187.204:ssh     rrcs-72-43-115-18:50615 ESTABLISHED
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  5      [ ]         DGRAM                    6559     /dev/log
unix  3      [ ]         STREAM     CONNECTED     9386     
unix  3      [ ]         STREAM     CONNECTED     9385     
. . .
```

Если вы добавите `-a` опцию, в ней будут перечислены все порты, прослушивающие и неподслушивающие:

```console
$ netstat -a
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 *:ssh                   *:*                     LISTEN     
tcp        0      0 192.241.187.204:ssh     rrcs-72-43-115-18:50615 ESTABLISHED
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN     
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ACC ]     STREAM     LISTENING     6195     @/com/ubuntu/upstart
unix  2      [ ACC ]     STREAM     LISTENING     7762     /var/run/acpid.socket
unix  2      [ ACC ]     STREAM     LISTENING     6503     /var/run/dbus/system_bus_socket
. . .
```

Если вы хотите отфильтровать только TCP или UDP соединения, используйте флаги -t или -u соответственно:

```
$ netstat -at
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 *:ssh                   *:*                     LISTEN     
tcp        0      0 192.241.187.204:ssh     rrcs-72-43-115-18:50615 ESTABLISHED
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
```

Смотрите статистику, установив флаг “-s”:

```console
$ netstat -s
Ip:
    13500 total packets received
    0 forwarded
    0 incoming packets discarded
    13500 incoming packets delivered
    3078 requests sent out
    16 dropped because of missing route
Icmp:
    41 ICMP messages received
    0 input ICMP message failed.
    ICMP input histogram:
        echo requests: 1
        echo replies: 40
. . .
```

Список всех активных прослушивающих портов UNIX с использованием netstat -lx.

```console
$ netstat -lx

Active UNIX domain sockets (only servers)
Proto RefCnt Flags Type State I-Node Path
unix 2 [ ACC ] STREAM LISTENING 4171 @ISCSIADM_ABSTRACT_NAMESPACE
unix 2 [ ACC ] STREAM LISTENING 5767 /var/run/cups/cups.sock
```

#### Отображение беспорядочного режима

Отображая беспорядочный режим с помощью переключателя -ac, netstat печатает выбранную информацию или обновляет экран каждые пять секунд. Экран по умолчанию обновляется каждую секунду.
Если вы хотите постоянно обновлять выходные данные, вы можете использовать флаг -c. В netstat доступно множество других опций, о которых вы узнаете, просмотрев его страницу руководства.

```console
$ netstat -ac 5 | grep tcp

tcp 0 0 *:sunrpc *:* LISTEN
tcp 0 0 *:58642 *:* LISTEN
tcp 0 0 *:ssh *:* LISTEN
```

#### Отображение IP-маршрутизации ядра

Отобразить таблицу маршрутизации IP ядра с помощью netstat и команды route.

```console
$ netstat -r

Kernel IP routing table
Destination Gateway Genmask Flags MSS Window irtt Iface
192.168.0.0 * 255.255.255.0 U 0 0 0 eth0
link-local * 255.255.0.0 U 0 0 0 eth0
default 192.168.0.1 0.0.0.0 UG 0 0 0 eth0
```

#### Анализ активных сетевых соединений и процессов, связанных с протоколом HTTP

```
netstat -ap | grep http
```

## Как настроить приоритеты процессов

Значения Nice могут варьироваться от -19 /-20 (наивысший приоритет) до 19/20 (низший приоритет) в зависимости от системы.

Чтобы запустить программу с определенным значением nice, вы можете использовать команду nice:

```
nice -n 15 command_to_execute
```

Это работает только при запуске новой программы.

Чтобы изменить значение nice в программе, которая уже выполняется, вы используете инструмент под названием renice:

```
renice 0 PID_to_prioritize
```

## Примеры использования `kill`

Если программа ведет себя неправильно и не завершает работу при указании термина signal, вы можете усилить сигнал, передав KILL signal:

```
kill -KILL PID_of_target_process
```

Например, многие процессы, предназначенные для постоянной работы в фоновом режиме (иногда называемые “демонами”), автоматически перезапускаются при получении сигнала HUP или зависания. Веб-сервер Apache обычно работает таким образом.

```
sudo kill -HUP pid_of_apache
```

Вы можете перечислить все сигналы , которые можно отправлять с помощью kill флага -l:

```
kill -l
```

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

Команда pkill работает почти точно так же, как kill, но вместо этого она оперирует именем процесса:

```
pkill -9 ping
```

Приведенная выше команда эквивалентна:

```
kill -9 `pgrep ping` 
```

Если вы хотите отправить сигнал каждому экземпляру определенного процесса, вы можете использовать команду killall:

```
killall firefox
```

## Примеры использования `ps`

### Просмотр всех запущенных процессов в различных форматах

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ ps -A
    PID TTY          TIME CMD
      1 ?        00:01:25 systemd
      2 ?        00:00:02 kthreadd
      3 ?        00:00:00 rcu_gp
```

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ ps -e | tail
 854266 ?        00:00:00 kworker/u40:2-events_unbound
 854320 ?        00:00:04 yandex_browser
 854444 ?        00:00:00 kworker/0:2
```

### Просмотр процессов, связанных с терминалом

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ ps -T
    PID    SPID TTY          TIME CMD
 804620  804620 pts/8    00:00:00 bash
 854641  854641 pts/8    00:00:00 ps
```

### Просмотр процессов, не связанных с терминалом

```console
panov@apanov-Legion-S7-16IAH7:~/Downloads$ ps -a
    PID TTY          TIME CMD
   2299 tty2     1-06:42:55 Xorg
   2465 tty2     00:00:00 gnome-session-b
   9583 pts/3    00:00:00 ssh
```

### Показать все текущие запущенные процессы

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ ps -ax
    PID TTY      STAT   TIME COMMAND
      1 ?        Ss     1:25 /lib/systemd/systemd --system --deserialize 49 splash
      2 ?        S      0:02 [kthreadd]
      3 ?        I<     0:00 [rcu_gp]
```

### Отображать все процессы в формате BSD

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0 167364 11040 ?        Ss    2023   1:25 /lib/systemd/systemd --system --deserialize 49 splash
root           2  0.0  0.0      0     0 ?        S     2023   0:02 [kthreadd]
root           3  0.0  0.0      0     0 ?        I<    2023   0:00 [rcu_gp]
root           4  0.0  0.0      0     0 ?        I<    2023   0:00 [rcu_par_gp]
root           5  0.0  0.0      0     0 ?        I<    2023   0:00 [slub_flushwq]
root           6  0.0  0.0      0     0 ?        I<    2023   0:00 [netns]
```

### Для выполнения полноформатного листинга

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ ps -eF
UID          PID    PPID  C    SZ   RSS PSR STIME TTY          TIME CMD
root           1       0  0 41841 11040  10  2023 ?        00:01:25 /lib/systemd/systemd --system --deserialize 49 splash
root           2       0  0     0     0   8  2023 ?        00:00:02 [kthreadd]
root           3       2  0     0     0   0  2023 ?        00:00:00 [rcu_gp]
```

### Фильтруйте процессы в соответствии с пользователем

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ ps -u apanov
    PID TTY          TIME CMD
   2117 ?        00:00:08 systemd
   2118 ?        00:00:00 (sd-pam)
   2124 ?        00:00:00 pipewire
   2125 ?        00:00:02 pipewire-media-
```

### Фильтровать процесс по потоку

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ ps -L 2117
    PID     LWP TTY      STAT   TIME COMMAND
   2117    2117 ?        Ss     0:08 /lib/systemd/systemd --user
```

### Показать каждый процесс, запущенный от имени root

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ ps -U root -u root
    PID TTY          TIME CMD
      1 ?        00:01:25 systemd
      2 ?        00:00:02 kthreadd
```

### Групповые процессы отображения

Если вы хотите составить список всех процессов, связанных с определенной группой, запустите
`ps -fG group_name` Или `ps -fG groupID` Например `ps -fG root`

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ ps -fG root
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0  2023 ?        00:01:25 /lib/systemd/systemd --system --deserialize 49 splash
root           2       0  0  2023 ?        00:00:02 [kthreadd]
root           3       2  0  2023 ?        00:00:00 [rcu_gp]
```

### PID процесса поиска

Скорее всего, вы обычно не знаете PID процесса. Вы можете выполнить поиск PID процесса, выполнив
`ps -C process_name` Например `ps -C bash`

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ ps -C bash
    PID TTY          TIME CMD
   9200 pts/0    00:00:00 bash
   9384 pts/3    00:00:00 bash
```

### Список процессов по PID

Вы можете отображать процессы по их PID, как показано на рисунке
`ps -fp PID` Например `ps -fp 1294`

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ ps -fp 9200
UID          PID    PPID  C STIME TTY          TIME CMD
apanov      9200    2913  0  2023 pts/0    00:00:00 /bin/bash
```

### Для отображения иерархии процессов в виде древовидной диаграммы

Обычно большинство процессов разветвляются на родительские процессы. Знакомство с этой родительско-дочерней связью может оказаться полезным. Приведенная ниже команда выполняет поиск процессов с именем apache2 `ps -f --forest -C bash`

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ ps -f --forest -C bash
UID          PID    PPID  C STIME TTY          TIME CMD
apanov    800047  799773  0 янв11 pts/2 00:00:00 /bin/bash --rcfile /snap/phpstorm/368/plugins/terminal/shell-integrations/bash/bash-integration.bash -i
root      762660  762659  0 янв10 pts/1 00:00:00 -bash
apanov    804620    2913  0 янв11 pts/8 00:00:00 /bin/bash
apanov    445828    2913  0  2023 pts/22   00:00:00 /bin/bash
apanov    445059    2913  0  2023 pts/21   00:00:00 /bin/bash
```

### Отображение дочерних процессов родительского процесса

Например, если вы хотите отобразить все разветвленные процессы, принадлежащие apache, выполните
`ps -o pid,uname,comm -C bash`

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ ps -o pid,uname,comm -C bash
    PID USER     COMMAND
   9200 apanov   bash
   9384 apanov   bash
   9585 apanov   bash
```

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ ps --ppid 9200
    PID TTY          TIME CMD
  26799 pts/0    1-09:04:02 htop
```

### Отображение потоков процесса

Команда ps может использоваться для просмотра потоков вместе с процессами. Приведенная ниже команда отображает все потоки, принадлежащие процессу, с идентификатором pid_no
`ps -p pid_no -L` Например `ps -p 1294 -L`

### Отображение выбранного списка столбцов

Вы можете использовать команду ps для отображения только необходимых вам столбцов. Например ,
`ps -e -o pid,uname,pcpu,pmem,comm`

### Переименование меток столбцов

Чтобы переименовать метки столбцов, выполните приведенную ниже команду
`ps -e -o pid=PID,uname=USERNAME,pcpu=CPU_USAGE,pmem=%MEM,comm=COMMAND`

### Отображение прошедшего времени процессов

Прошедшее время указывает на то, как долго выполнялся процесс
`ps -e -o pid,comm,etime`

```console
apanov@apanov-Legion-S7-16IAH7:~/Downloads$ ps -e -o pid,comm,etime
    PID COMMAND             ELAPSED
      1 systemd         56-02:04:02
      2 kthreadd        56-02:04:02
      3 rcu_gp          56-02:04:02
```

### Использование команды ps с grep

команда ps может использоваться вместе с командой grep для поиска определенного процесса, например
`ps -ef  | grep systemd`


