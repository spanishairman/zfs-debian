# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.
  config.vm.define "Debian12" do |srv|

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
    srv.vm.box = "/home/max/vagrant/images/debian12"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  #  config.vm.network "private_network", ip: "192.168.33.10"
  # config.vm.network "private_network", ip: "192.168.33.10", adapter: "5", type: "ip"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.
  # system("virsh net-create /home/max/vagrant/vagrant-libvirt-mgmt.xml")
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
  config.vm.provision "shell", inline: <<-SHELL
    brd='*************************************************************'
  # echo "$brd"
  # echo 'Если ранее не были установлены, то добавим репозитории contrib и установим необходимые  пакеты'
  # echo "$brd"
  # sed -r -i'.BAK' 's/^deb(.*)$/deb\1 contrib/g' /etc/apt/sources.list
  # sed -r -i'.BAK' 's/main/main contrib/' /etc/apt/sources.list
  # apt update
  # apt install -y rsync arch-install-scripts linux-headers-amd64 zfsutils-linux zfs-dkms zfs-zed
    echo "$brd"
    echo 'Создадим пулы raid1'
    echo "$brd"
    zpool create data1 mirror /dev/vdb /dev/vdc
    zpool create data2 mirror /dev/vdd /dev/vde
    zpool create data3 mirror /dev/vdf /dev/vdg
    zpool create data4 mirror /dev/vdh /dev/vdi
    zfs set compression=lzjb data1
    zfs set compression=lz4 data2
    zfs set compression=gzip-9 data3
    zfs set compression=zle data4
    zfs get all | grep compression
  # Все необходимые для раьботы файлы скачаны заранее в образ машины, из которого далее выполнен импорт бокса.
  # Поэтому команды загрузки закрываем комментарием.
  #  wget -P /home/vagrant https://gutenberg.org/cache/epub/2600/pg2600.converter.log 
    echo 'Скопируем скачаный файл журнала на наши пулы с разными алгоритмами сжатия. После этого сравним их эффективность'
    for i in {1..4}; do cp /home/vagrant/pg2600.converter.log /data$i; done
    echo 'Выводим список файлов'
    ls -lh /data*
    echo 'Список файловых систем'
    zfs list
    echo 'Распакуем архив с экспортированным пулом'
    tar -xzvf /home/vagrant/archive.tar.gz
    echo 'Проверим возможность импорта'
    zpool import -d zpoolexport/
    echo 'Импортируем'
    zpool import -d zpoolexport/ otus
    echo 'Выводим параметры zpool и zfs'
    zpool get all otus
    zfs get all otus
    echo 'Восстанавливаем файловую систему из снапшота'
    zfs receive otus/test@today < /home/vagrant/otus_task2.file
    echo 'Ищем в каталоге /otus/test файл с именем “secret_message”'
    find /otus/test -name "secret_message"
    echo 'Выводим содержимое'
    cat /otus/test/task1/file_mess/secret_message
    SHELL
  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
  end

  # config.vm.define "ArchLinux1" do |srv1|
  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
    # srv1.vm.box = "/home/max/vagrant/images/debian12"
  # config.vm.provider "libvirt" do |lv|
  #   lv.memory = "1024"
  #   lv.cpu = "2"
  # end
  # end
end
