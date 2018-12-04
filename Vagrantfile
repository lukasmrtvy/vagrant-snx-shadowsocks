VAGRANT_BOX = 'ubuntu/cosmic64'
VM_NAME = 'vagrantbox'

Vagrant.configure(2) do |config|
  config.vm.box = VAGRANT_BOX
  config.vm.hostname = VM_NAME
  config.vm.provider "virtualbox" do |v|
    v.name = VM_NAME
    v.memory = 2048
  end
  config.vm.network "forwarded_port", guest: 80, host: 8080, auto_correct: true
  config.vm.network "forwarded_port", guest: 443, host: 4443, auto_correct: true

  config.vm.network "forwarded_port", guest: 8388, host: 8388, protocol: "tcp", host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 8388, host: 8388, protocol: "udp", host_ip: "127.0.0.1"

  #config.vm.network "private_network", type: "dhcp"

  config.vm.usable_port_range = 8000..8999

  config.vm.synced_folder "shared", "/shared", create: true, automount: true

  config.vm.provision :shell, :inline  => '

   curl -sSL -k https://vpnportal.aktifbank.com.tr/SNX/INSTALL/snx_install.sh -o /tmp/snx_install.sh

   echo iptables-persistent iptables-persistent/autosave_v4 boolean true | sudo debconf-set-selections
   echo iptables-persistent iptables-persistent/autosave_v6 boolean true | sudo debconf-set-selections

   dpkg --add-architecture i386

   apt-get update  && apt-get install -y \
      shadowsocks-libev \
      iptables-persistent \
      libnss-resolve \
      libnss-resolve:i386 \
      libpam0g:i386 \
      libx11-6:i386 \
      libstdc++6:i386 \
      libstdc++5:i386

   bash /tmp/snx_install.sh

   iptables -t nat -A OUTPUT -p tcp -d X.X.X.X --dport 443 -j DNAT --to-destination Y.Y.Y.Y

   /sbin/iptables-save > /etc/iptables/rules.v4

   sed -i "s|DAEMON_ARGS=.*|DAEMON_ARGS=\"-s 0.0.0.0 -u --fast-open -k secret \"|g" /etc/default/shadowsocks-libev

   systemctl restart shadowsocks-libev.service

   echo "shared /shared vboxsf defaults 0 0" >> /etc/fstab

   unlink /etc/resolv.conf
   rm -f /etc/resolv.conf
   echo "nameserver 127.0.0.53" > /etc/resolv.conf

   timedatectl set-timezone Europe/Prague
  '
end
