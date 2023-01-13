# mdadm
raid
В репозитории vagrant file, с конфигом создания ВМ.
Должна создаться ВМ 
Name – raids
ОЗУ – 1024
CPU – 1 
Dick – 1 основной и 4-е доп диска по 2Гб
Загрузится ПО «mdadm,  gdisk, parted, smartctl»

Соберётся 5-й raid

Создадим раздел GPT на raid.
parted -s /dev/md0 mklabel gpt

Создадим партиции 
parted /dev/md0 mkpart primary ext4 0% 20%
parted /dev/md0 mkpart primary ext4 20% 40%
parted /dev/md0 mkpart primary ext4 40% 60%
parted /dev/md0 mkpart primary ext4 60% 80%
parted /dev/md0 mkpart primary ext4 80% 100%

создадим ФС на созданный партициях
for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done

монтируем
mkdir -p /raid/part{1,2,3,4,5}
for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; don 

Также удалось произвести замену диска в RAID
# вытащить диск, сначало метим его.
mdadm --manage /dev/md0 --fail /dev/sdc 
# вытащить диск из масива
mdadm --manage /dev/md0 --remove /dev/sdc
# добавляем диск
mdadm --manage /dev/md0 --add /dev/sde
