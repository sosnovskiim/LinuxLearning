# Продолжение настройки ssh
Пользовательские настройки подключения хранятся в файле `~/.ssh/config` в домашней папке пользователя.
Системные настройки подключения хранятся в файле `/etc/openssh/ssh_config`.
Они применяются по умолчанию для всех пользователей системы, если не переопределены в пользовательском файле `~/.ssh/config`.

Судя по всему, в данном контексте файлом options является `~/.ssh/config`.
Чтобы можно было подключаться к хосту, не вводя имя пользователя и порт, в данный файл необходимо добавить следующее:
```
Host student
        HostName 95.31.204.147
        User student
        Port 230
```
Теперь попробуем подключиться к указанному хосту.
```
ssh student
```
![1.png](/solutions/SSH-2/screenshots/1.png)
