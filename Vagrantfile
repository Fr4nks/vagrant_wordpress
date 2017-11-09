# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
vagrantConfig = YAML.load_file '../Vagrantfile.config.yml'

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.network "private_network", ip: vagrantConfig['ip']
  config.vm.synced_folder vagrantConfig['synced_folder']['host_path'], vagrantConfig['synced_folder']['guest_path'], 
    owner: nil, 
    group: nil,
    nfs: true,
    nfs_version: 4,
    nfs_udp: false,
    linux__nfs_options: ['rw','no_subtree_check','all_squash','async']

  config.vm.provider "virtualbox" do |vb|     
    vb.memory = "2048"
    vb.cpus = 2
    # Uncomment option below to avoid issues with VirtualBox on Windows 10
    # vb.gui = true

  end
  
  config.vm.provision "file", source: "~/.gitconfig", destination: ".gitconfig"
  
  #Install php 
  config.vm.provision "shell", inline: <<-SHELL
    echo PHP INSTALL
    sudo apt-get -y update
    sudo add-apt-repository ppa:ondrej/php
    sudo apt-get -y update
    sudo apt-get install -y php7.0 libapache2-mod-php7.0 php7.0 php7.0-common php7.0-gd php7.0-mysql php7.0-mcrypt php7.0-curl php7.0-intl php7.0-xsl php7.0-mbstring php7.0-zip php7.0-bcmath php7.0-iconv php7.0-soap

    echo PHP PATCH
    file="/etc/php/7.0/cli/php.ini"

    echo APACHE2 PATCH
    sudo apt-get install -y apache2
    sudo update-rc.d apache2 defaults
    sudo a2enmod rewrite

    file="/etc/apache2/sites-available/000-default.conf"
    sudo sed -i "s~<\/VirtualHost>~ <Directory "\/var\/www\/html\">\\n    AllowOverride All\\n  <\/Directory>\\n&~" ${file}
    sudo service apache2 restart

    sudo apt-get -y install mysql-server
    sudo update-rc.d mysql defaults
    sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password #{vagrantConfig['mysql']['password']}'
    sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password #{vagrantConfig['mysql']['password']}'

    echo CREATE DATABASE
    sudo mysql --user=#{vagrantConfig['mysql']['username']} --password=#{vagrantConfig['mysql']['password']} -e \"CREATE DATABASE #{vagrantConfig['magento']['db_name']};\"

    echo INSTALL COMPOSER
    curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
    mv composer.phar /usr/local/bin/composer
    composer clearcache
    composer global config github-oauth.github.com #{vagrantConfig['github_oauth']['github_com']}
    composer clearcache
    
    echo INSTALL GIT
    sudo apt-get install -y git

    echo CLEAN UP FOLDERS
    sudo rm -Rf /var/www/html
    sudo rm -rf #{vagrantConfig['synced_folder']['guest_path']}.* 2> /dev/null
    sudo ln -s #{vagrantConfig['synced_folder']['guest_path']} /var/www/html
    
    echo DOWNLOAD WORDPRESS
    wget https://wordpress.org/latest.tar.gz #{vagrantConfig['synced_folder']['guest_path']}
    tar -xzvf #{vagrantConfig['synced_folder']['guest_path']}latest.tar.gz
     
    echo GIT CLONE MY THEME 
    sudo git clone https://github.com/Fr4nks/EndTag #{vagrantConfig['synced_folder']['guest_path']}app/design/frontend/EndTag

    echo INSTALL GRUNT
    sudo apt-get install -y nodejs
    sudo apt-get install -y npm
    sudo ln -fs /usr/bin/nodejs /usr/local/bin/node
    
    sudo npm install -g grunt-cli
    echo RENAME GRUNTFILE.JS.SAMPLE to GRUNTFILE.JS
    mv #{vagrantConfig['synced_folder']['guest_path']}Gruntfile.js.sample #{vagrantConfig['synced_folder']['guest_path']}Gruntfile.js

SHELL
end
