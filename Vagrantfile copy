Vagrant.configure("2") do |config|
  # Base configuration
  config.vm.box = "bento/ubuntu-20.04"
  config.vm.hostname = "docker-host"

  # Network configuration for apps
  config.vm.network "forwarded_port", guest: 8080, host: 8080  # Vote app
  config.vm.network "forwarded_port", guest: 8888, host: 8888  # Result app

  # VMware configuration
  config.vm.provider "vmware_workstation" do |vmw|
    vmw.gui = true
    vmw.vmx["memsize"] = "4096"
    vmw.vmx["numvcpus"] = "2"
  end

  # Sync project directory
  config.vm.synced_folder ".", "/home/vagrant/docker-project"

  # Setup Docker and users
  config.vm.provision "shell", inline: <<-SHELL
    # Install Docker and Docker Compose
    curl -fsSL https://get.docker.com -o get-docker.sh
    sh get-docker.sh
    curl -L "https://github.com/docker/compose/releases/download/v2.20.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose

    # Set passwords
    echo "root:root" | chpasswd
    echo "vagrant:vagrant" | chpasswd
    
    # Add vagrant user to docker group
    usermod -aG docker vagrant
  SHELL

  # Start Docker Compose
  config.vm.provision "shell", inline: <<-SHELL, run: "always"
    cd /home/vagrant/docker-project
    docker-compose up -d
  SHELL
end
