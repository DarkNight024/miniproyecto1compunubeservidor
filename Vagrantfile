# -*- mode: ruby -*-
# vi: set ft=ruby :

# Script para instalar Consul
$script = <<SCRIPT
echo "Installing dependencies ..."
sudo apt-get update
sudo apt-get install -y unzip curl jq dnsutils vim git

echo "Adding HashiCorp GPG key and repository..."
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install consul

# Configurar Consul en el servidor
if [[ "$(hostname)" == "servidor" ]]; then

    echo "Installing Haproxy..."
    sudo apt install haproxy -y

    echo "Configuring Consul on the server..."
    sudo consul agent -ui -server -client=0.0.0.0 -bootstrap-expect=3 -node=agent-server \
    -bind=192.168.100.10 -data-dir=/tmp/consul -retry-join="192.168.100.10" -retry-join="192.168.100.11" \
    -retry-join="192.168.100.12" > /dev/null 2>&1 &

    echo "Configuring Haproxy on the server..."
    sudo cp /home/vagrant/data/haproxy.cfg /etc/haproxy/haproxy.cfg
    sudo cp /home/vagrant/data/503p.http /etc/haproxy/errors/503p.http 
    sudo systemctl restart haproxy

    echo "Instalando node y npm"
    sudo apt-get install -y nodejs
    sudo apt-get install -y npm

    echo "Installing artillery.io on the server..."
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
    source ~/.bashrc
    nvm install 18.16.1
    nvm use 18.16.1
    npm install -g artillery@latest
    sudo npm install -g -y artillery@latest

    echo "Adding test.ylm..."
    sudo cp /home/vagrant/data/test.yml /home/vagrant/test.yml

fi

# Configurar Consul en n1
if [[ "$(hostname)" == "n1" ]]; then
    echo "Configuring Consul on the server..."
    sudo consul agent -ui -server -client=0.0.0.0 -node=agent-n1 \
    -bind=192.168.100.11 -data-dir=/tmp/consul -retry-join="192.168.100.10" -retry-join="192.168.100.11" \
    -retry-join="192.168.100.12" > /dev/null 2>&1 &

    echo "Instalando node y npm"
    sudo apt-get install -y nodejs
    sudo apt-get install -y npm

    echo "Clonar github"
    git clone https://github.com/omondragon/consulService

    sudo sed -i "s/const HOST='192.168.100.3';/const HOST='192.168.100.11';/" consulService/app/index.js
    cd consulService/app && npm install consul express && node index.js 3000 &
fi

# Configurar Consul en n2
if [[ "$(hostname)" == "n2" ]]; then
    echo "Configuring Consul on the server..."
    sudo consul agent -ui -server -client=0.0.0.0 -node=agent-n2 \
    -bind=192.168.100.12 -data-dir=/tmp/consul -retry-join="192.168.100.10" -retry-join="192.168.100.11" \
    -retry-join="192.168.100.12" > /dev/null 2>&1 &

    echo "Instalando node y npm"
    sudo apt-get install -y nodejs
    sudo apt-get install -y npm

    echo "Clonar github"
    git clone https://github.com/omondragon/consulService
    
    sudo sed -i "s/const HOST='192.168.100.3';/const HOST='192.168.100.12';/" consulService/app/index.js
    cd consulService/app && npm install consul express && node index.js 3001 &
fi

SCRIPT

Vagrant.configure("2") do |config|

  if Vagrant.has_plugin? "vagrant-vbguest"
    config.vbguest.no_install = true
    config.vbguest.auto_update = false
    config.vbguest.no_remote = true
    config.vm.box_download_options = {"ssl-revoke-best-effort" => true}
  end

  config.vm.box = "bento/ubuntu-22.04"

  config.vm.provision "shell", inline: $script

  config.vm.define "servidor" do |servidor|
    servidor.vm.hostname = "servidor"
    servidor.vm.network "private_network", ip: "192.168.100.10"
    servidor.vm.synced_folder "C:\\Users\\Sebastian\\OneDrive\\Escritorio\\sites\\miniproyecto1\\servidor", "/home/vagrant/data"
  end  
  
  config.vm.define "n1" do |n1|
      n1.vm.hostname = "n1"
      n1.vm.network "private_network", ip: "192.168.100.11"
      n1.vm.synced_folder "C:\\Users\\Sebastian\\OneDrive\\Escritorio\\sites\\miniproyecto1\\n1", "/home/vagrant/data"
  end

  config.vm.define "n2" do |n2|
      n2.vm.hostname = "n2"
      n2.vm.network "private_network", ip: "192.168.100.12"
      n2.vm.synced_folder "C:\\Users\\Sebastian\\OneDrive\\Escritorio\\sites\\miniproyecto1\\n2", "/home/vagrant/data"
  end

end