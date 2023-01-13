# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
  :raids => {
    :box_name => "centos/7",
    :box_version => "1804.02",
    :disks => {
      :sata1 => {
        :dfile => home + '/VirtualBox VMs/sata1.vdi',
        :size => 2048,
        :port => 1
      },
      :sata2 => {
        :dfile => home + '/VirtualBox VMs/sata2.vdi',
        :size => 2048, # Megabytes
        :port => 2
      },
      :sata3 => {
        :dfile => home + '/VirtualBox VMs/sata3.vdi',
        :size => 2048, # Megabytes
        :port => 3
      },
      :sata4 => {
        :dfile => home + '/VirtualBox VMs/sata4.vdi',
        :size => 2048,
        :port => 4
      }
    }
  },
}

Vagrant.configure("2") do |config|

    config.vm.box_version = "1804.02"

    MACHINES.each do |boxname, boxconfig|
config.ssh.insert_key = false
        config.vm.define boxname do |box|

            box.vm.box = boxconfig[:box_name]
            box.vm.host_name = boxname.to_s

            box.vm.provider :virtualbox do |vb|
                vb.customize ["modifyvm", :id, "--memory", "1024"]
                needsController = false
                boxconfig[:disks].each do |dname, dconf|
                    unless File.exist?(dconf[:dfile])
                        vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                        needsController =  true
                    end

                end

                if needsController == true
                    vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                    boxconfig[:disks].each do |dname, dconf|
                        vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                    end
                end

            end

            box.vm.provision "shell", inline: <<-SHELL
              mkdir -p ~root/.ssh
              cp ~vagrant/.ssh/auth* ~root/.ssh
              yum install -y mdadm  gdisk parted smartctl
              mdadm --create --verbose /dev/md0 -l 5 -n 3 /dev/sd{b,d,c}
              mkdir /etc/mdadm
              echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
              mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
              #mkfs.ext4 /dev/md0
              #mount /dev/md0 /mnt
              parted -s /dev/md0 mklabel gpt
              parted /dev/md0 mkpart primary ext4 0% 20%
              parted /dev/md0 mkpart primary ext4 20% 40%
              parted /dev/md0 mkpart primary ext4 40% 60%
              parted /dev/md0 mkpart primary ext4 60% 80%
              parted /dev/md0 mkpart primary ext4 80% 100%

              for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done

              mkdir -p /raid/part{1,2,3,4,5}
              for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done
            SHELL

        end
    end
end
