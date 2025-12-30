# Пишем юниты

1. Создайте скрипт который создаёт папку заполняет её файлами ( имена 1-4 ) и записывает в них информацию
о текущей дате, версии ядра, имени компьютера и списе всех файлов в домашнем каталоге пользователя от которого выполняется скрипт( не забудьте сдлеать проверку на существование файлов и папок)
```
#!/usr/bin/env bash
set -euo pipefail

# 1. Создать папку, если её нет
mkdir -p "$WORKDIR"

# 2. Проверить наличие файлов и при необходимости создавать
[ -f "$WORKDIR/1" ] || touch "$WORKDIR/1"
[ -f "$WORKDIR/2" ] || touch "$WORKDIR/2"
[ -f "$WORKDIR/3" ] || touch "$WORKDIR/3"
[ -f "$WORKDIR/4" ] || touch "$WORKDIR/4"

# 3. Заполнить файлы информацией
# 1 — текущая дата
date > "$WORKDIR/1"

# 2 — версия ядра
uname -r > "$WORKDIR/2"

# 3 — имя компьютера
hostname > "$WORKDIR/3"

# 4 — список всех файлов в домашнем каталоге пользователя
find "$HOME" -maxdepth 1 -mindepth 1 -printf '%f\n' > "$WORKDIR/4"
```
Делаем исполняемым:<br>
`chmod +x ~/unit_task.sh`
2. Создайте юнит который будет вызывать этот скрипт при запуске. Проверьте<br>
Файл юнита `/etc/systemd/system/unit-task.service`:
```
[Unit]
Description=Run unit_task script at boot

[Service]
Type=oneshot
ExecStart=/home/andrey/unit_task.sh

[Install]
WantedBy=multi-user.target
```
Активация и проверка:<br>
```
sudo systemctl daemon-reload
sudo systemctl start unit-task.service
sudo systemctl status unit-task.service
```
3. Создайте таймер который будет вызывать выполнение одноимённого systemd юнита каждые 5 минут.<br>
Файл таймера `/etc/systemd/system/unit-task.timer`:<br>
```
[Unit]
Description=Run unit_task service every 5 minutes

[Timer]
OnBootSec=5min
OnUnitActiveSec=5min
Unit=unit-task.service

[Install]
WantedBy=timers.target
```
Включим таймер:<br>
```
sudo systemctl daemon-reload
sudo systemctl enable --now unit-task.timer
sudo systemctl list-timers unit-task.timer
```
4. От какого пользователя вызыаются юниты поумолчанию?<br>
Для системных юнитов (/etc/systemd/system, /usr/lib/systemd/system) по умолчанию процессы запускаются от пользователя root, если в юните явно не указан User=
5. Создайте пользователя от имени которого будет выполняться ваш скрипт.<br>
`sudo useradd -m -s /bin/bash unituser`
Скопировать скрипт этому пользователю:<br>
```
sudo cp ~/unit_task.sh /home/unituser/unit_task.sh
sudo chown unituser:unituser /home/unituser/unit_task.sh
sudo chmod +x /home/unituser/unit_task.sh
```
6. Дополните юнит информацией о пользователе от которого должен выплняться скрипт.
Обновлённый `/etc/systemd/system/unit-task.service`:
```
[Unit]
Description=Run unit_task script at boot as unituser

[Service]
Type=oneshot
User=unituser
WorkingDirectory=/home/unituser
ExecStart=/home/unituser/unit_task.sh

[Install]
WantedBy=multi-user.target
```
Обновление конфигурации и проверка:<br>
```
sudo systemctl daemon-reload
sudo systemctl restart unit-task.service
sudo systemctl status unit-task.service
```
User=unituser указывает, от какого пользователя будет запускаться скрипт; WorkingDirectory= задаёт рабочий каталог.
7. Дополните ваш скрипт так, что бы он независимо от местоположения всега выполнялся в домашней папке того кто его вызывает.
```
cd "$HOME"
WORKDIR="$HOME/unit_files"
```
$HOME всегда указывает на домашний каталог пользователя, под которым запущен процесс, поэтому скрипт работает в нужной домашней директории независимо от того, из какого каталога он был вызван.