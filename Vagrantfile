Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"

  # Install Docker and Docker Compose
  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
    sudo apt-get update
    sudo apt-get install -y docker-ce docker-compose
  SHELL

  # Sync project directory to VM
  config.vm.synced_folder ".", "/home/vagrant/docker-project"

  # Deploy Docker containers using docker-compose
  config.vm.provision "shell", inline: <<-SHELL
    cd /home/vagrant/docker-project/Docker-Cats-Dogs && sudo docker-compose up -d
  SHELL

  # Configure VMware Workstation provider
  config.vm.provider "vmware_desktop" do |v|
    v.vmx["memsize"] = "2048"   # Adjust memory if needed
    v.vmx["numvcpus"] = "2"     # Adjust CPU count if needed
  end
end
