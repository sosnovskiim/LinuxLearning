# RAID-массивы
**RAID** (redundant array of independent disks — избыточный массив независимых дисков) –
технология виртуального хранения данных, объединяющая несколько физических дисков в один логический элемент.
В зависимости от выбранной спецификации RAID, могут быть повышены скорость чтения/записи и/или уровень защищенности от потери данных.

Сущетсвуют аппаратные и программные RAID-массивы:
- Программные создаются уже после установки операционной системы
средствами программных продуктов и утилит, что и является главным недостатком таких дисковых массивов.
- Аппаратные создают дисковый массив до установки операционной системы и от неё не зависят.

Типы RAID-массивов:
- **RAID 1** (также называют «Mirror» – Зеркало) предполагает полное дублирование данных с одного физического диска на другой.
К недостаткам RAID 1 можно отнести то, что вы получаете в два раза меньше дискового пространства.
Т.е. если вы используете 2 диска по 256 Гб, то система будет видеть всего 1 размером 256 Гб.
Данный вид RAID не дает выигрыша в скорости, но значительно повышает уровень отказоустойчивости,
ведь если один диск выйдет из строя, всегда есть его полная копия. Запись и стирание с дисков происходит одновременно.
Если информация была намеренно удалена, то возможности восстановить её с другого диска уже не будет.

- **RAID 0** (также называют «Striping» – Чередование) предполагает разделение информации на блоки и одновременная запись разных блоков на разные диски.
Такая технология повышает скорость чтения/записи, позволяет пользователю использовать полный суммарный объем дисков,
однако понижает отказоустойчивость, вернее сводит её на ноль. Так, в случае выхода из строя одного из дисков,
восстановить информацию будет практически невозможно. Для сборки RAID 0 рекомендуется использовать исключительно высоконадежные диски.

- **RAID 5** можно назвать более усовершенствованным RAID 0. Можно использовать от 3 жестких дисков.
На все, кроме одного, записывается RAID 0, а на последний специальная контрольная сумма,
что позволяет сохранить информацию на винчестерах в случае «смерти» одного из них (но не более одного).
Скорость работы такого массива высокая. На восстановление информации в случае замены диска потребуется много времени.

- **RAID 10** является миксом RAID 1 и 0. И объединяет в себе плюсы от каждого: высокая производительность и высокая отказоустойчивость.
Массив обязательно содержит четное количество дисков (минимум 4) и является самым надежным вариантом сохранения информации.
Недостатком является высокая стоимость дискового массива: эффективная емкость составит половину от общей емкости дискового пространства.

- **RAID 50** является миксом RAID 5 и 0. Строится RAID 5, но его составляющими будут не самостоятельные жесткие диски, а массивы RAID 0.

## Работа с RAID-массивами

В виртуальную машину добавляем еще один диск, повторно выполняя действия из задания "Монитрование диска".
Убедимся в успешном добавлении с помощью команды `df` с флагом, отображающим только файловую систему ext4.
```
df -t ext4
```
![1.png](/solutions/Files-4/screenshots/1.png)

Теперь отмонтируем оба диска и создадим массив RAID 0 с помощью утилиты `mdadm`. Значение частей команды:
- `--create` - создание RAID-массива;
- `/dev/md0` - название создаваемого массива;
- `--level=0` - тип RAID-массива;
- `--raid-devices=2` - количество дисков в массиве;
- `/dev/sda1 /dev/sdc1` - перечисление конкретных дисков.
```
mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/sda1 /dev/sdc1
```
![2.png](/solutions/Files-4/screenshots/2.png)

Форматируем файловую систему с ext2 на ext4 для созданного RAID-массива.
```
mkfs -t ext4 /dev/md0
```
![3.png](/solutions/Files-4/screenshots/3.png)

Монтируем RAID-массив и проверяем его работоспособность.
```
mount /dev/md0 /mnt/test; lsblk
```
![4.png](/solutions/Files-4/screenshots/4.png)
```
mdadm --detail /dev/md0
```
![5.png](/solutions/Files-4/screenshots/5.png)
```
cd /mnt/test; touch file1 file2 file3; ls
```
![6.png](/solutions/Files-4/screenshots/6.png)

Отмонтируем RAID-массив и остановим его. Затем необходимо затереть superblock каждого из составляющих массива,
иначе система будет автоматически пытаться восстановить массив при запуске.
```
umount /dev/md0
mdadm --stop /dev/md0
mdadm --zero-superblock --force /dev/sda1
mdadm --zero-superblock --force /dev/sdc1
```
Перезапускаем систему и проверяем, что массив удален.
```
lsblk
```
![7.png](/solutions/Files-4/screenshots/7.png)

Аналогично создаем массив RAID 1, форматируем файловую систему с ext2 на ext4,
монтируем и проверяем его работоспособность.
```
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
```
![8.png](/solutions/Files-4/screenshots/8.png)
```
mkfs -t ext4 /dev/md0
```
![9.png](/solutions/Files-4/screenshots/9.png)
```
mount /dev/md0 /mnt/test; lsblk
```
![10.png](/solutions/Files-4/screenshots/10.png)
```
mdadm --detail /dev/md0
```
![11.png](/solutions/Files-4/screenshots/11.png)

## Различия RAID 0 и 1 
Различия созданных массивов заключаются в том, что объем RAID 0 равен сумме объемов дисков,
из которых он состоит, а размер RAID 1 не изменился, так как содержимое дисков дублируется.

## Файловые системы, поддерживающие RAID-массивы без сторонних утилит
- Btrfs - позволяет создавать массивы RAID 0, RAID 1 и RAID 10.
- ZFS - позволяет создавать массивы RAID-Z и зеркала, аналогичные RAID 5 и RAID 1 соответственно.

## Создание RAID-массива во время установки системы
В Alt Linux доступно создание RAID-массива во время установки системы.

Необходимо создать раздел типа RAID.

![12.png](/solutions/Files-4/screenshots/12.png)

А затем создать RAID-массив нужного типа в данном разделе.

![13.png](/solutions/Files-4/screenshots/13.png)
