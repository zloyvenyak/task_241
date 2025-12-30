# Юниты

1. Что такое systemd юнит?<br>
Systemd-юнит (unit) — это объект конфигурации systemd, описывающий ресурс/задачу, которой systemd управляет (например, service, mount, socket, target, timer и т.д.).<br>
Юниты обычно задаются файлами с расширениями вроде .service, .mount, .socket, .target, .timer.
2. Проверье статус любого systemd юнита, какую информацию выводит эта кманда?<br>
Проверим sshd:<br>
`systemctl status sshd.service`<br>
![img.png](img.png)<br>
Команда `systemctl status` выводит состояние юнита(loaded/active/sub), информацию о процессе(PID), а также последние строки журнала, связанные с этим юнитом.
3. ПОпробуйте оставновить сервис.<br>
`sudo systemctl stop sshd.service`<br>
![img_1.png](img_1.png)
4. Перезапустите его.<br>
`sudo systemctl restart sshd.service`<br>
![img_2.png](img_2.png)
5. УДалите из автозагрузки<br>
`sudo systemctl disable sshd.service`<br>
![img_3.png](img_3.png)
6. Верните обратно<br>
`sudo systemctl enable sshd.service`<br>
![img_4.png](img_4.png)
7. Что такое таймеры?<br>
Systemd timers — это юниты с расширением .timer, которые позволяют планировать запуск других юнитов (обычно .service) по времени/событиям, как замена cron-подобному расписанию.<br>
Таймер-юнит управляется systemd и при срабатывании активирует связанный service-юнит.
