### Дисковая подсистема. ZFS
#### Подготовка окружения
В нашем примере используется гипервизор Qemu-KVM, библиотека Libvirt. В качестве хостовой системы - OpenSuse Leap 15.5.

Для работы Vagrant с Libvirt установлен пакет vagrant-libvirt:
```
Сведения — пакет vagrant-libvirt:
---------------------------------
Репозиторий            : Основной репозиторий
Имя                    : vagrant-libvirt
Версия                 : 0.10.2-bp155.1.19
Архитектура            : x86_64
Поставщик              : openSUSE
Размер после установки : 658,3 KiB
Установлено            : Да
Состояние              : актуален
Пакет с исходным кодом : vagrant-libvirt-0.10.2-bp155.1.19.src
Адрес источника        : https://github.com/vagrant-libvirt/vagrant-libvirt
Заключение             : Провайдер Vagrant для libvirt
Описание               : 

    This is a Vagrant plugin that adds a Libvirt provider to Vagrant, allowing
    Vagrant to control and provision machines via the Libvirt toolkit.
```
Пакет Vagrant также устанавливаем из репозиториев. Текущая версия для OpenSuse Leap 15.5:
```
max@localhost:~/vagrant/vg3> vagrant -v
Vagrant 2.2.18
```
Образ операционной системы создан заранее, для этого установлен [Debian Linux из официального образа netinst](https://www.debian.org/distrib/netinst)

#### Подготовка базового образа
Подготовим образ для работы. Для этого добавим репозиторий ***contrib*** в файл /etc/apt/sources.list:
```
root@debian12:~# sed -r -i'.BAK' 's/^deb(.*)$/deb\1 contrib/g' /etc/apt/sources.list
```
или так:
```
root@debian12:~# sed -r -i'.BAK' 'main/main contrib' /etc/apt/sources.list
```
Обновим список пакетов, а также все установленные пакеты:
```
root@debian12:~# apt update && apt upgrade
```
Установим необходимые нам:
```
root@debian12:~# apt install linux-headers-amd64 zfsutils-linux zfs-dkms zfs-zed rsync lvm2 arch-install-scripts
root@debian12:~# apt autoremove
```
Скачиваем все, необходимые для выполнения работы, файлы:
```
vagrant@debian12:~$ wget https://gutenberg.org/cache/epub/2600/pg2600.converter.log
vagrant@debian12:~$ wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download'
vagrant@debian12:~$ wget -O otus_task2.file --no-check-certificate 'https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download'
```
Выключаем виртуальную сашину:
```
root@debian12:~# shutdown -P now
```
#### Сборка нового бокса на базе обновленного образа виртуальной машины
Создаем новый бокс:
```
localhost:~ # rm /home/max/vagrant/images/debian12
localhost:~ # rm -rf /home/max/.vagrant.d
localhost:~ # cd /home/max/vagrant
localhost:~ # ./create-box.sh /home/max/libvirt/images/debian12.qcow2 /home/max/vagrant/images/debian12 vagrantfile.info

```
Здесь используются - скрипт [create-box.sh](create-box.sh) и файл с параметрами [vagrantfile.info](vagrantfile.info)

Следующий блок [Vagrantfile](Vagrantfile) отвечает за оперативную память и набор дополнительных дисков в виртуальной машине:
```
  config.vm.provider "libvirt" do |lv|
    lv.memory = "3072"
    lv.cpus = "2"
    lv.title = "Debian12"
    lv.description = "Виртуальная машина на базе дистрибутива Debian Linux"
    lv.management_network_name = "vagrant-libvirt-mgmt"
    lv.management_network_address = "192.168.121.0/24"
    lv.management_network_keep = "true"
    lv.management_network_mac = "52:54:00:27:28:83"
    lv.storage :file, :size => '1G', :device => 'vdb', :allow_existing => false
    lv.storage :file, :size => '1G', :device => 'vdc', :allow_existing => false
    lv.storage :file, :size => '1G', :device => 'vdd', :allow_existing => false
    lv.storage :file, :size => '1G', :device => 'vde', :allow_existing => false
    lv.storage :file, :size => '1G', :device => 'vdf', :allow_existing => false
    lv.storage :file, :size => '1G', :device => 'vdg', :allow_existing => false
    lv.storage :file, :size => '1G', :device => 'vdh', :allow_existing => false
    lv.storage :file, :size => '1G', :device => 'vdi', :allow_existing => false
  end
```
#### Создание пулов ***zpool*** и файловых систем ***zfs***
Создаём пулы и задаем разные методы сжатия на них:
```
    zpool create data1 mirror /dev/vdb /dev/vdc
    zpool create data2 mirror /dev/vdd /dev/vde
    zpool create data3 mirror /dev/vdf /dev/vdg
    zpool create data4 mirror /dev/vdh /dev/vdi
    zfs set compression=lzjb data1
    zfs set compression=lz4 data2
    zfs set compression=gzip-9 data3
    zfs set compression=zle data4
    zfs get all | grep compression
```
#### Работа с фафйловыми системами, тестирование, сравнение
Скопируем скачаный файл журнала на наши пулы с разными алгоритмами сжатия. После этого сравним их эффективность:
```
    for i in {1..4}; do cp /home/vagrant/pg2600.converter.log /data$i; done
    echo 'Выводим список файлов'
    ls -lh /data*
    echo 'Список файловых систем'
    zfs list
```
Распакуем архив с экспортированным пулом
```
    tar -xzvf /home/vagrant/archive.tar.gz
```
Проверим возможность импорта
```
    zpool import -d zpoolexport/
```
Импортируем
```
    zpool import -d zpoolexport/ otus
```
Выводим параметры zpool и zfs'
```
    zpool get all otus
    zfs get all otus
```
Восстанавливаем файловую систему из снапшота
```
    zfs receive otus/test@today < /home/vagrant/otus_task2.file
```
Ищем в каталоге /otus/test файл с именем “secret_message”
```
    find /otus/test -name "secret_message"
```
Выводим содержимое
```
    cat /otus/test/task1/file_mess/secret_message
```
Готово!
