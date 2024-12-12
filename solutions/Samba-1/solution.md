# Общие папки
Если в дистрибутиве по умолчанию нет samba, то устанавливаем пакет из репозитория: `apt-get install samba`.
После установки создаем резервную копию файла конфигурации: `cp /etc/samba/smb.conf /etc/samba/smb.conf_sample`.

**Samba** – серверное приложение, реализующее доступ клиентских терминалов к папкам, принтерам и дискам про протоколу SMB/CIFS.
В частности, используется для общего доступа к папкам и файлам в локальной сети.

Создаем папку от root `mkdir -p /media/samba/test1` и меняем права доступа к ней только на чтение для всех пользователей, кроме root, `chmod 744 /media/samba/test1`.

Далее добавим в файл `/etc/samba/smb.conf` следующий код:
```
[test1]
    path = /media/samba/test1
    public = yes
    browsable = yes
    writable = no
    read only = yes
    guest ok = yes
```
