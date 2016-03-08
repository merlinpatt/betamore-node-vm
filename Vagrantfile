# -*- mode: ruby -*-
# vi: set ft=ruby :

ports = [8000]
app_name = "betamore"

Vagrant.configure(2) do |config|
  config.vm.box = "https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-i386-vagrant-disk1.box"
  config.vm.hostname = app_name
  config.vm.box_check_update = false
  config.ssh.insert_key = false

  ports.each do |port|
    config.vm.network "forwarded_port", guest: port, host: port
  end

  config.vm.synced_folder "../", "/#{app_name}"

  config.vm.provider "virtualbox" do |vb|
    vb.name = app_name
  end

  config.vm.provision "shell" do |shell|
    shell.inline = <<-SHELL

    echo "Installing Git"
      apt-get install -y git >> /vagrant/install.log 2>> /vagrant/error.log
    echo "Git installed"

    echo "Installing Redis"
      apt-get install -y redis-server >> /vagrant/install.log 2>> /vagrant/error.log
    echo "Redis installed"

    echo "Installing Node"
      curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash - >> /vagrant/install.log 2>> /vagrant/error.log
      apt-get install -y nodejs >> /vagrant/install.log 2>> /vagrant/error.log
    echo "Node installed"

    echo "Installing Postgres"
      apt-get install -y postgresql-9.3 >> /vagrant/install.log 2>> /vagrant/error.log

      conf=pg_hba.conf
      touch $conf
      echo "local all postgres peer" > $conf
      echo "local all all peer" >> $conf
      echo "host all all 127.0.0.1/32 trust" >> $conf
      echo "host all all ::1/128 md5" >> $conf
      mv $conf "/etc/postgresql/9.3/main/pg_hba.conf"

      service postgresql restart >> /vagrant/install.log 2>> /vagrant/error.log
      sudo -u postgres createdb fswd_lab_5_development
    echo "Postgres installed"

    SHELL
  end

  config.vm.provision "shell" do |shell|
    shell.inline = <<-SHELL
    echo "------"
    echo "Install finished. Remove ./install.log if there were no errors."
    echo "------"
    SHELL
  end


  config.vm.provision "node_module mounting", type: "shell", run: "always" do |shell|
    shell.privileged = false
    shell.args = "'#{app_name}'"
    shell.inline = <<-SHELL
    echo "Mounting node_modules directories"
      for directory in /$1/*; do
        if [ -f $directory/package.json ]; then
          mkdir -p $directory/node_modules
          mkdir -p /home/vagrant$directory/node_modules
          sudo mount --bind /home/vagrant$directory/node_modules $directory/node_modules
        fi
      done
    echo "Directories created"
    echo "-------"
    echo "cd into /$1 to see your projects"
    SHELL
  end
end
