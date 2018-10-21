VAGRANT_BOX = 'ubuntu/bionic64'
VM_NAME = 'vagrantbox'

Vagrant.configure(2) do |config|
  config.vm.box = VAGRANT_BOX
  config.vm.hostname = VM_NAME
  config.vm.provider "virtualbox" do |v|
    v.name = VM_NAME
    v.memory = 2048
  end


  config.vm.network "forwarded_port", guest: 1080, host: 1080, protocol: "tcp", host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 1080, host: 1080, protocol: "udp", host_ip: "127.0.0.1"

  config.vm.network "private_network", type: "dhcp"

  config.vm.usable_port_range = 8000..8999

  config.vm.synced_folder "shared", "/shared", create: true

  config.vm.provision :shell, :inline  => '

    curl -sSL -k https://vpnportal.aktifbank.com.tr/SNX/INSTALL/snx_install.sh -o /tmp/snx_install.sh

    echo iptables-persistent iptables-persistent/autosave_v4 boolean true | sudo debconf-set-selections
    echo iptables-persistent iptables-persistent/autosave_v6 boolean true | sudo debconf-set-selections

    dpkg --add-architecture i386

    apt -qq update && apt -qq upgrade -y && apt -qq install -y \
    shadowsocks-libev \
    resolvconf \
    iptables-persistent \
	  libpam0g:i386 \
	  libx11-6:i386 \
	  libstdc++6:i386 \
	  ibstdc++5:i386

	sudo bash /tmp/snx_install.sh

	# iptables -t nat -A OUTPUT -p tcp -d 10.250.30.33 --dport 443 -j DNAT --to-destination X.X.X.X
	/sbin/iptables-save > /etc/iptables/rules

	sudo cat <<- EOF >> /lib/systemd/system/shadowsocks-libev-local@config.service
    [Unit]
    Description=Shadowsocks-Libev Custom Client Service for %I
    Documentation=man:ss-local(1)
    After=network.target

    [Service]
    Type=simple
    CapabilityBoundingSet=CAP_NET_BIND_SERVICE
    ExecStart=/usr/bin/ss-local -b 0.0.0.0 -c /etc/shadowsocks-libev/%i.json

    [Install]
    WantedBy=multi-user.target
EOF

    sed -i "s|DAEMON_ARGS=.*|DAEMON_ARGS=\"-u --fast-open\"|g" /etc/default/shadowsocks-libev

    sudo systemctl restart shadowsocks-libev.service

    sudo systemctl start shadowsocks-libev-local@config.service
    sudo systemctl enable shadowsocks-libev-local@config.service

  '
  
end
