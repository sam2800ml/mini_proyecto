# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|



  if Vagrant.has_plugin? "vagrant-vbguest"
    config.vbguest.no_install  = true
    config.vbguest.auto_update = false
    config.vbguest.no_remote   = true
  end

  
  config.vm.box = "bento/ubuntu-22.04"
  config.vm.synced_folder "nodedata", "/home/vagrant/nodedata"
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get -y upgrade

    cd nodedata
    sudo apt install -y nodejs
    node -v
    sudo apt install -y npm
    npm -v
    sudo npm install consul
    sudo npm install express
  SHELL

  config.vm.define :haproxys do |haproxys|
    haproxys.vm.network :private_network, ip: "192.168.56.10"
    haproxys.vm.hostname = "haproxys"
    haproxys.vm.provision "shell", inline: <<-SHELL
    sudo apt install haproxy -y
    sudo systemctl enable haproxy
    haproxy -v
    cd ../../etc/haproxy
    cp /home/vagrant/nodedata/haproxy.cfg haproxy.cfg
    cd
    sudo systemctl restart haproxy
    SHELL
  end

  config.vm.define :server1 do |server1|
    server1.vm.network :private_network, ip: "192.168.56.11"
    server1.vm.hostname = "server1"
    #server1.vm.synced_folder ".", "nodedata"
    server1.vm.provision "shell", inline: <<-SHELL
    wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
    echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
    sudo apt update && sudo apt install consul
    consul -v
    nohup consul agent -ui -server -bootstrap-expect=1 -node=leader -bind=192.168.56.11 -client=0.0.0.0 -data-dir=. -enable-script-checks=true &
    sleep 10
    cd nodedata
    nohup node indexleader.js 80 &
    sleep 5
    SHELL
  end

  config.vm.define :server2 do |server2|
    server2.vm.network :private_network, ip: "192.168.56.12"
    server2.vm.hostname = "server2"
    server2.vm.provision "shell", inline: <<-SHELL
    wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
    echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
    sudo apt update && sudo apt install consul
    consul -v
    nohup consul agent -ui -node=agent -bind=192.168.56.12 -client=0.0.0.0 -data-dir=. -enable-script-checks=true &
    sleep 10
    consul join 192.168.56.11
    cd nodedata
    nohup node index.js 80 &
    sleep 5
    SHELL
    
  end

end
