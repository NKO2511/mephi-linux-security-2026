# ДЗ №1. Изучение средств защиты ОС GNU/Linux

Студент: Никита Костяев, уникальный номер **M265876**

ОС: Fedora Linux 44 (Workstation Edition), виртуальная машина Oracle VirtualBox

## Выполнение заданий

| Раздел | Задание | Как выполнено | Подтверждение |
|---|---|---|---|
| 1 | Пользователь `user1` с UID 1234 в группе `students`, смена пароля каждые 3 месяца | `groupadd students`<br>`useradd -u 1234 -G students -m user1`<br>`passwd user1`<br>`chage -M 90 user1` | `passwd`: `user1:x:1234:1234::/home/user1:/bin/bash`<br>`group`: `students:x:1001:user1`<br>`shadow`: максимальный срок действия пароля `90` |
| 2.1 | Поиск файлов с битом set-UID | `find / -perm -4000 -type f -fprintf ~/suid_files.out '%#m %u %p\n'` | `suid_files.out` |
| 2.2 | Процессы с EUID = 0 и RUID обычного пользователя | В одном терминале `user1` запустил `passwd`, во втором:<br>`ps -eo pid,ruid,ruser,euid,euser,cmd \| awk 'NR==1 \|\| ($4==0 && $2>=1000)'` | `proc_monitor.out`: процесс `passwd` с RUID `1234 (user1)` и EUID `0 (root)` |
| 3 | Привилегированная операция через set-UID | Утилита `cat`, операция — чтение `/etc/shadow`:<br>`cp /usr/bin/cat /home/user1/cat_suid`<br>`chown root:root /home/user1/cat_suid`<br>`chmod u+s /home/user1/cat_suid`<br>Проверка от `user1`: `cat /etc/shadow` — «Отказано в доступе», `./cat_suid /etc/shadow` — успешно | `stat.out`: `cat_suid` — `4755/-rwsr-xr-x`, владелец `root` |
| 4 | Привилегированная операция через механизм привилегий (capabilities) | Утилита `chown`, операция — смена владельца файла на `root`:<br>`cp /usr/bin/chown /home/user1/chown_cap`<br>`chown user1:user1 /home/user1/chown_cap`<br>`setcap cap_chown=ep /home/user1/chown_cap`<br>Проверка от `user1`: `chown root testfile` — «Операция не позволена», `./chown_cap root:root testfile` — успешно | `getcap.out`: `/home/user1/chown_cap cap_chown=ep`<br>`stat.out`: `chown_cap` — `0755`, владелец `user1` |
| 5 | Разрешить `user1` изменять системное время через sudo | В `/etc/sudoers` через `visudo` добавлено правило:<br>`user1 ALL=(root) /usr/bin/date, /usr/bin/timedatectl`<br>Проверка: `visudo -c` — успешно; от `user1`: `sudo -l`, `sudo date -s "2026-11-01 12:00:00"` — успешно | `sudoers` |
| 6 | Сбор артефактов | Скриншот терминала с уникальным номером; сохранены история команд и вывод `stat`, `getcap`, команд мониторинга | `mephi-screenshot.png`, `history.out`, `history_user1.out`, `stat.out`, `getcap.out`, `suid_files.out`, `proc_monitor.out` |
| 7 | Публикация на GitHub | Публичный репозиторий `mephi-linux-security-2026` | Этот репозиторий |

Все настройки (пользователь, группа, срок действия пароля, бит set-UID, привилегии файла, правило sudo) хранятся в файлах и атрибутах файловой системы, поэтому сохраняются после перезагрузки.

## Состав репозитория

| Файл | Раздел | Содержимое |
|---|---|---|
| `mephi-screenshot.png` | 6 | Скриншот терминала с уникальным номером |
| `history.out` | Все | История команд администратора (`nikikstv`) |
| `history_user1.out` | 2–5 | История команд пользователя `user1` |
| `stat.out` | 1, 3, 4 | `stat` для `/home/user1`, `cat_suid`, `chown_cap` |
| `getcap.out` | 4 | Привилегии `chown_cap` |
| `passwd`, `shadow`, `group` | 1 | Копии `/etc/passwd`, `/etc/shadow`, `/etc/group` |
| `sudoers` | 5 | Копия `/etc/sudoers` |
| `suid_files.out` | 2.1 | Файлы с битом set-UID |
| `proc_monitor.out` | 2.2 | Процессы с повышенными привилегиями |
