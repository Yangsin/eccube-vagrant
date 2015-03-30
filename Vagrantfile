﻿# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

$shell = <<SCRIPT
## Timezone の設定
sudo cp -p  /usr/share/zoneinfo/Japan /etc/localtime
## Database の作成
sudo -u postgres createuser -d -S -R eccube_db_user
sudo -u postgres createdb -U eccube_db_user -E utf-8 eccube_db
### optional #############################
## 以下のコメントを有効にすると, remote_db のコピーをローカルに作成する
#
# sudo -u vagrant echo "remote_host:5432:remote_db:remote_db_user:password" > .pgpass
# chown vagrant:vagrant .pgpass
# chmod 600 .pgpass
# sudo -u vagrant pg_dump -h remote_host -U remote_db_user remote_db | psql -U test_db_user test_db
### END optional #########################

echo 'Congratulations!!! Install Success.'
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "puppetlabs/centos-6.6-32-nocm"

  config.vm.network "private_network", ip: "192.168.33.10"

  config.vm.synced_folder "../ec-cube3", "/ec-cube",
    :create => true,
    :owner=> 'vagrant', :group=>'vagrant',
    :mount_options => ["dmode=777,fmode=777"]

  config.vm.provision :chef_solo do |chef|
    #chef.log_level = :debug
    chef.cookbooks_path = ["./chef/cookbooks", "./chef/site-cookbooks"]
    chef.roles_path = "./chef/roles"
    chef.data_bags_path = "./chef/data_bags"

    chef.add_recipe     "iptables::disabled"
    chef.add_recipe     "apache2"
    chef.add_recipe     "apache2::mod_php5"
    chef.add_recipe     "apache2::mod_ssl"
    chef.add_recipe     "apache2::mod_rewrite"
    chef.add_recipe     "php"
    chef.add_recipe     "postgresql::client"
    chef.add_recipe     "postgresql::server"
    chef.add_recipe     "ec-cube3"

    chef.json = {
      :apache => {
        :version => "2.2",
        :default_site_enabled => true,
        :docroot_dir => "/ec-cube/html",
        :listen_ports => [80, 443]
      },
      :php => {
        :version => "5.3",
        :directives => {
          :display_errors => 'On',
          "date.timezone" => "Asia/Tokyo",
        },
        :packages => ["php-mbstring", "php-pdo", "php-pgsql", "php-mysql", "php-pear", "php-xml", "php-gd", "php-soap", "php-devel"]
      },
      :postgresql => {
        :password => {
          postgres: 'password'
        },
        :pg_hba => [
          {:type => 'local', :db => 'all', :user => 'postgres', :addr => nil, :method => 'trust'},
          {:type => 'local', :db => 'all', :user => 'all', :addr => nil, :method => 'trust'},
          {:type => 'host', :db => 'all', :user => 'all', :addr => '127.0.0.1/32', :method => 'trust'},
          {:type => 'host', :db => 'all', :user => 'all', :addr => '::1/128', :method => 'trust'}
        ]
      }
    }
  end

  config.omnibus.chef_version = :latest
  # config.berkshelf.enabled = true

  config.vm.provision "shell", inline: $shell
end