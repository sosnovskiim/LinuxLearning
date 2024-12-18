# Написание пользовательских юнитов
Создадим файл в каталоге командой `touch /usr/local/bin/script`.
Изменим права доступа к файлу, чтобы его можно было выполнить, командой `chmod +x`.
Теперь откроем данный файл текстовым редактором, например `nano`.
Далее разместим в файле следующий код:
```bash
#!/bin/bash
set -euo pipefail

HOME="$(eval echo "~$(whoami)")"

# Создание папки и перемещение в нее
cd
if ! [ -d info ]; then
  mkdir info
fi
cd info

# Создание первого файла и сохранение текущей даты в него
date > 1

# Создание второго файла и сохранение версии ядра в него
uname -r > 2

# Создание третьего файла и сохранение имени компьютера в него
uname -n > 3

# Создание четвертого файла и сохранение списка всех файлов
# в домашнем каталоге пользователя, от которого выполняется скрипт
cd
files="$(ls -l)"
cd info
echo "$files" > 4
```

## Создание юнита
Создадим файл в каталоге, в котором обычно хранятся пользовательские юниты, командой `touch /etc/systemd/system/script.service`.
Теперь откроем данный файл текстовым редактором, например `nano`.
Далее разместим в файле следующий код:
```
[Unit]
Description=SomeService

[Service]
Type=forking
ExecStart=/bin/bash /usr/local/bin/script
User=sosnovskiim

[Install]
WantedBy=multi-user.target
```
В секции **[Unit]** описывается, что это за юнит. В данном случае здесь содержится только описания юнита.

В секции **[Service]** указывается какими командами и под каким пользователем должен запускаться сервис.
Переменной `Type` присваивается значение `forking`. В таком случае systemd предполагает,
что служба запускается однократно и процесс разветвляется с завершением родительского процесса.
Данный тип используется для запуска классических демонов.
Переменной `ExecStart` присваивается значение, указывающее полный путь к скрипту, который необходимо запустить.
В переменной `User` указывается, от имени какого пользователя должен стартоваться сервис.
По умолчанию все общесистемные юниты запускаются с правами суперпользователя.

В секции **[Install]** описывается, на каком уровне запуска должен стартовать сервис.
Переменной `WantedBy` присваивается значение `multi-user.target`, что соответствует третьему уровню запуска системы:
"Многопользовательский режим без графики".

Теперь добавляем юнит в автозапуск, проверяем его статус, и перезагружаем systemd, чтобы применить изменения.
```
systemctl enable script
systemctl status script
systemctl daemon-reload
```
![1.png](/solutions/Systemd-2/screenshots/1.png)

Перезапускаем систему, проверяем статус юнита и содержимое созданных файлов.
```
systemctl status script
cat info/1
cat info/2
cat info/3
cat info/4
```
![2.png](/solutions/Systemd-2/screenshots/2.png)

## Создание таймера
Создадим файл с тем же именем, что и ранее добавленный юнит, но с другим расширением,
в каталоге, в котором обычно хранятся пользовательские юниты, командой `touch /etc/systemd/system/script.timer`.
Теперь откроем данный файл текстовым редактором, например `nano`.
Далее разместим в файле следующий код:
```
[Unit]
Description=SomeTimer

[Timer]
OnBootSec=1m
OnUnitActiveSec=2m

[Install]
WantedBy=timers.target
```
В секции [Timer] указываются условия запуска таймера. Переменная `OnBootSec` содержит время после запуска системы,
через которое будет стартовать юнит. Переменная `OnUnitActiveSec` содержит периодичность, с которой юнит будет автоматически запускаться.

Теперь добавляем таймер в автозапуск, проверяем его статус, и перезагружаем systemd, чтобы применить изменения.
```
systemctl enable script.timer
systemctl status script.timer
systemctl daemon-reload
```
![3.png](/solutions/Systemd-2/screenshots/3.png)

Перезапускаем систему, проверяем статус таймера и содержимое файла с текущей датой.
Затем ждем автоматического перезапуска скрипта и снова проверяем статус таймера и изменения в файле с датой.
```
systemctl status script.timer
cat info/1
```
![4.png](/solutions/Systemd-2/screenshots/4.png)
