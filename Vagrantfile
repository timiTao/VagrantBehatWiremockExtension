# -*- mode: ruby -*-
# vi: set ft=ruby :

PROJECT_PATH = ENV['PROJECT_PATH'] || "./"

Vagrant.configure(2) do |config|

  config.vm.box = "chef/centos-7.0"
  config.vm.box_check_update = false

  config.berkshelf.enabled = true
  #config.omnibus.chef_version = :latest

  config.vm.provider "virtualbox" do |vb|
     # Display the VirtualBox GUI when booting the machine
     # vb.gui = true
     vb.memory = "512"
   end
  
  config.vm.define "server", primary: true do |server|
    server.vm.hostname = "server.vagrant"
    server.vm.post_up_message = "Vagrant server is set up"

    server.vm.network "private_network", ip: "192.168.205.10"

    server.vm.synced_folder PROJECT_PATH, "/home/vagrant/project"

    # Chef solo provisioning
    server.vm.provision "chef_solo" do |chef|
      chef.add_recipe "php"
      chef.json = {
        :php => {
            :configure_options => [
                "-enable-mbstring",
                "--with-zlib",
                "--with-pear",
                "--enable-ftp",
                "--with-gd",
                "--with-freetype-dir",
                "--with-jpeg-dir=",
                "--enable-gd-native-ttf",
                "--with-gettext",
                "--with-mcrypt",
                "--enable-pcntl",
                "--enable-soap",
                "--enable-sockets",
                "--with-xmlrpc",
                "--with-xsl",
                "--enable-zip",
                "--enable-mbstring",
            ],
            :directives => {
                "date.timezone" => "Europe/Rome",
                "error_reporting" => "E_ALL",
                "display_errors" => "On"
            },
        },
      }
    end
    server.vm.provision "shell", 
        inline: "yum install php-mbstring -y"
  end

  (1..2).each do |i|
    config.vm.define "client-#{i}" do |client|
      client.vm.hostname = "client-#{i}.vagrant"
      client.vm.network "private_network", ip: "192.168.205.1#{i}"
      client.vm.provision "shell", inline: "echo client #{i} is CREATED"

      # Chef solo provisioning
      client.vm.provision "chef_solo" do |chef|
        chef.add_recipe "java"
      end

      client.vm.provision "shell", 
        inline: "wget http://repo1.maven.org/maven2/com/github/tomakehurst/wiremock/1.56/wiremock-1.56-standalone.jar"
      client.vm.provision "shell", 
        inline: "java -jar wiremock-1.56-standalone.jar --port 80 &", run: "always"
    end
  end
end
