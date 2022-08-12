# Домашнее задание
## Настраиваем бэкапы

Все дальнейшие действия были проверены при использовании
```
marozov@iMac-marozov ~ % vagrant -v
Vagrant 2.2.19
```
```
marozov@imac logs % vboxmanage -v
6.1.32r149290
```
CentOS Linux release 7.8 из Vagrant Cloud
```
[vagrant@log ~]$ cat /etc/redhat-release
CentOS Linux release 7.8.2003 (Core)
```

```
Настроить стенд Vagrant с двумя виртуальными машинами: backup_server и client.

Настроить удаленный бекап каталога /etc c сервера client при помощи borgbackup. Резервные копии должны соответствовать следующим критериям:

директория для резервных копий /var/backup. Это должна быть отдельная точка монтирования. В данном случае для демонстрации размер не принципиален, достаточно будет и 2GB;
репозиторий дле резервных копий должен быть зашифрован ключом или паролем - на ваше усмотрение;
имя бекапа должно содержать информацию о времени снятия бекапа;
глубина бекапа должна быть год, хранить можно по последней копии на конец месяца, кроме последних трех. Последние три месяца должны содержать копии на каждый день. Т.е. должна быть правильно настроена политика удаления старых бэкапов;
резервная копия снимается каждые 5 минут. Такой частый запуск в целях демонстрации;
написан скрипт для снятия резервных копий. Скрипт запускается из соответствующей Cron джобы, либо systemd timer-а - на ваше усмотрение;
настроено логирование процесса бекапа. Для упрощения можно весь вывод перенаправлять в logger с соответствующим тегом. Если настроите не в syslog, то обязательна ротация логов.
Запустите стенд на 30 минут. 
Убедитесь что резервные копии снимаются. 
Остановите бекап, удалите (или переместите) директорию /etc и восстановите ее из бекапа. 
Для сдачи домашнего задания ожидаем настроенные стенд, логи процесса бэкапа и описание процесса восстановления. 
Формат сдачи ДЗ - vagrant + ansible
```
** **
в вагранте создается отдельнвый vdi который монтируется для бэкапа.
SSH ключи автоматически отправляются на сервер через nfs шару.
для автоматизации использовал юнит и  таймер sustemd.
вывод в journald через logger.

получим список бэкапов
```
[vagrant@client ~]$ sudo borg list borg@backupserver:/var/backup/repo/client
Remote: Warning: Permanently added 'backupserver,192.168.10.10' (ECDSA) to the list of known hosts.
Enter passphrase for key ssh://borg@backupserver/var/backup/repo/client: 
```
удаляю директорию
```
rm -rf /etc/yum
```
извлечем удаленыый контент из поледнего бэкапап, посмотрим его содержимое
```
sudo borg list borg@backupserver:/var/backup/repo/client::client-
```
извлекаем
```
sudo borg extract borg@backupserver:/var/backup/repo/client::
```
результат
```
[vagrant@client ~]$ sudo ls etc
yum
```
