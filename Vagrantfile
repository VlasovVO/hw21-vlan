# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
:inetRouter => {
        :box_name => "centos/6",
        :net => [
                   {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                ]
  },
  :centralRouter => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                   {ip: '192.168.255.5', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "office1-net"},
                   {ip: '192.168.255.9', adapter: 4, netmask: "255.255.255.252", virtualbox__intnet: "office2-net"},
                   {ip: '192.168.0.1', adapter: 5, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {ip: '192.168.0.33', adapter: 6, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},
                   {ip: '192.168.0.65', adapter: 7, netmask: "255.255.255.192", virtualbox__intnet: "mgt-net"},
                ]
  },
  
  :centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {adapter: 3, auto_config: false, virtualbox__intnet: true},
                   {adapter: 4, auto_config: false, virtualbox__intnet: true},
                ]
  },
  
  :office1Router => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.6', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office1-net"},
                   {ip: '192.168.2.1', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "dev-net"},
                   {ip: '192.168.2.65', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "test-net"},
                   {ip: '192.168.2.129', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "mng-net"},
                   {ip: '192.168.2.193', adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "hw-net"},
                ]
  },
  :office1Server => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.2.66', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "test-net"},
                   {adapter: 3, auto_config: false, virtualbox__intnet: true},
                   {adapter: 4, auto_config: false, virtualbox__intnet: true},
                ]
  },
  :office2Router => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.10', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office2-net"},
                   {ip: '192.168.1.1', adapter: 3, netmask: "255.255.255.128", virtualbox__intnet: "dev-net"},
                   {ip: '192.168.1.129', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "test-net"},
                   {ip: '192.168.1.193', adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "hw-net"},
                ]
  },
  :office2Server => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.1.130', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "test-net"},
                   {adapter: 3, auto_config: false, virtualbox__intnet: true},
                   {adapter: 4, auto_config: false, virtualbox__intnet: true},
                ]
  },
  
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s

        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
        
        if boxconfig.key?(:public)
          box.vm.network "public_network", boxconfig[:public]
        end

        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
                cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL
        
        case boxname.to_s
        when "inetRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo net.ipv4.conf.all.forwarding=1 >> /etc/sysctl.d/99-sysctl.conf
            sudo iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
            sudo service network restart
            sudo ip rout add 192.168.0.0/28 via 192.168.255.2
            sudo ip rout add 192.168.1.128/26 via 192.168.255.2
            sudo ip rout add 192.168.1.2/26 via 192.168.255.2
            sudo ip rout add 192.168.255.4/30 via 192.168.255.2
            sudo ip rout add 192.168.255.8/30 via 192.168.255.2
            SHELL
        when "centralRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo net.ipv4.conf.all.forwarding=1 >> /etc/sysctl.d/99-sysctl.conf
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            sudo ip rout add 192.168.2.64/26 via 192.168.255.6
            sudo ip rout add 192.168.1.128/26 via 192.168.255.10
            SHELL
        when "office1Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo net.ipv4.conf.all.forwarding=1 >> /etc/sysctl.d/99-sysctl.conf
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.255.5" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            sudo ip rout add 192.168.255.0/30 via 192.168.255.5
            sudo ip rout add 192.168.0.0/28 via 192.168.255.5
            sudo ip rout add 192.168.1.128/26 via 192.168.255.5
            SHELL
        when "office2Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo net.ipv4.conf.all.forwarding=1 >> /etc/sysctl.d/99-sysctl.conf
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.255.9" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            sudo ip rout add 192.168.255.0/30 via 192.168.255.9
            sudo ip rout add 192.168.0.0/28 via 192.168.255.9
            sudo ip rout add 192.168.2.64/26 via 192.168.255.9
            SHELL
        when "centralServer"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            SHELL
        when "office1Server"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.2.65" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            SHELL
        when "office2Server"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.1.129" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            SHELL
        end
      end
  end  
end
