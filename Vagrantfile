# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'

# holder for the planets read from universe.d
universe = []

Dir['./universe.d/*.yml'].each do |planets_file|
    planets = YAML::load_file(planets_file)

    for planet in planets
        universe.push(planet)
    end

end

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    config.vm.box = "base"

    # virtualbox configuration
    config.vm.provider "virtualbox" do |v|
        v.name = "Mulkave Dev Environment"
    end

    # provision docker images & containers

    config.vm.provision "docker" do |d|

        # d.pull_images 'vinelab/base'

        # run containers
        for planet in universe

            # skip this planet, let's go to the next one where there are aliens and guns
            if planet.has_key?('skip') and planet['skip'] == true
                next
            end

            if planet.has_key?('docker')

                name     = planet['docker']['name']
                image    = planet['docker']['image']
                hostname = planet['docker']['hostname']
                volumes  = ''
                ports    = ''
                links    = ''

                # add volumes
                if planet['docker'].has_key?('volumes')

                    for volume in planet['docker']['volumes']
                        volumes += " -v #{volume['host']}:#{volume['guest']}"
                    end

                end

                # add ports
                if planet['docker'].has_key?('ports')

                    for port in planet['docker']['ports']

                        # we'll set the ports regardless of what
                        # data type used
                        case port
                        when Hash

                            ports += " -p #{port['host']}:#{port['guest']}"

                            if port.has_key?('expose') and port['expose'] == true
                                ports += " -p #{port['guest']}"
                            end

                        when Array
                            ports += " -p #{port[0]}:#{port[1]}"
                        else
                            ports += " -p #{port}"
                        end

                    end

                end

                # add links
                if planet['docker'].has_key?('links')

                    for link in planet['docker']['links']
                        links += " --link=#{link}"
                    end

                end
                puts "ports: #{ports}"
                # tell master what docker will be running
                puts "--Docker-- will Run: #{image} as #{name}"

                # tell vagrant to run dat container
                d.run name, image: image, args: "-h #{hostname} #{volumes} #{ports} #{links}"

            end
        end

    end

    # VM host-guest configuration
    for planet in universe

        # check and forward Vagrant ports
        if planet.has_key?('ports')
            for port in planet['ports']
                if port.class == Hash
                    config.vm.network :forwarded_port, guest: port['guest'], host: port['host']
                end
            end
        end

        # check and forward Docker ports
        if planet.has_key?('docker') and planet['docker'].has_key?('ports')
            for port in planet['docker']['ports']
                # this might seem a little bit confusing but
                # the idea behind it is that Vagrant will forward
                # and receive on the same port exposed by Docker
                if port.class == Hash
                    config.vm.network :forwarded_port, guest: port['host'], host: port['host']
                end
            end
        end

        # sync folders
        if planet.has_key?('sync')
            for sync in planet['sync']
                config.vm.synced_folder sync['host'], sync['guest'], :mount_options => ['dmode=777,fmode=777']
            end
        end

    end


    # Custom Configuration
    config.vm.network :forwarded_port, guest: 6022, host: 6022
    config.vm.network :forwarded_port, guest: 6023, host: 6023
    config.vm.network :forwarded_port, guest: 6024, host: 6024
    config.vm.network :forwarded_port, guest: 6025, host: 6025
    config.vm.network :forwarded_port, guest: 8529, host: 8529

    # config.vm.network "forwarded_port", guest: 8022, host: 8022 # webservers

    # config.vm.define 'web-redis' do |wr|
    #
    #     wr.vm.provision "docker" do |d|
    #
    #         d.build_image "/vagrant/docker"
    #
    #     end
    #
    #     wr.vm.provider "virtualbox" do |v|
    #         v.name = "docker"
    #     end
    #
    #   # cluster
    #   wr.vm.network "forwarded_port", guest: 8022, host: 8022 # webservers
    #   wr.vm.network "forwarded_port", guest: 8122, host: 8122 # databases
    #   wr.vm.network "forwarded_port", guest: 8080, host: 8080 # http 80
    #
    #   # all in one
    #   wr.vm.network "forwarded_port", guest: 8822, host: 8822 # server
    #   wr.vm.network "forwarded_port", guest: 8880, host: 8880 # http 80
    #
    #   wr.vm.synced_folder "./app", "/home/vagrant/app", :mount_options => ["dmode=777,fmode=777"]
    #
    # end
    #
    # config.vm.define 'spv', primary: true do |s|
    #     s.vm.provision "docker" do |d|
    #         d.pull_images "centos"
    #     end
    #
    #     s.vm.provider "virtualbox" do |v|
    #         v.name = "spv-docker"
    #     end
    # end

  #
  # config.vm.provision "ansible" do |a|
  #    a.playbook = "vagrant.yml"
  # end


  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  # config.vm.box_url = "http://domain.com/path/to/above.box"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  # config.ssh.forward_agent = true

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Don't boot with headless mode
  #   vb.gui = true
  #
  #   # Use VBoxManage to customize the VM. For example to change memory:
  #   vb.customize ["modifyvm", :id, "--memory", "1024"]
  # end
  #
  # View the documentation for the provider you're using for more
  # information on available options.

  # Enable provisioning with Puppet stand alone.  Puppet manifests
  # are contained in a directory path relative to this Vagrantfile.
  # You will need to create the manifests directory and a manifest in
  # the file base.pp in the manifests_path directory.
  #
  # An example Puppet manifest to provision the message of the day:
  #
  # # group { "puppet":
  # #   ensure => "present",
  # # }
  # #
  # # File { owner => 0, group => 0, mode => 0644 }
  # #
  # # file { '/etc/motd':
  # #   content => "Welcome to your Vagrant-built virtual machine!
  # #               Managed by Puppet.\n"
  # # }
  #
  # config.vm.provision "puppet" do |puppet|
  #   puppet.manifests_path = "manifests"
  #   puppet.manifest_file  = "site.pp"
  # end

  # Enable provisioning with chef solo, specifying a cookbooks path, roles
  # path, and data_bags path (all relative to this Vagrantfile), and adding
  # some recipes and/or roles.
  #
  # config.vm.provision "chef_solo" do |chef|
  #   chef.cookbooks_path = "../my-recipes/cookbooks"
  #   chef.roles_path = "../my-recipes/roles"
  #   chef.data_bags_path = "../my-recipes/data_bags"
  #   chef.add_recipe "mysql"
  #   chef.add_role "web"
  #
  #   # You may also specify custom JSON attributes:
  #   chef.json = { :mysql_password => "foo" }
  # end

  # Enable provisioning with chef server, specifying the chef server URL,
  # and the path to the validation key (relative to this Vagrantfile).
  #
  # The Opscode Platform uses HTTPS. Substitute your organization for
  # ORGNAME in the URL and validation key.
  #
  # If you have your own Chef Server, use the appropriate URL, which may be
  # HTTP instead of HTTPS depending on your configuration. Also change the
  # validation key to validation.pem.
  #
  # config.vm.provision "chef_client" do |chef|
  #   chef.chef_server_url = "https://api.opscode.com/organizations/ORGNAME"
  #   chef.validation_key_path = "ORGNAME-validator.pem"
  # end
  #
  # If you're using the Opscode platform, your validator client is
  # ORGNAME-validator, replacing ORGNAME with your organization name.
  #
  # If you have your own Chef Server, the default validation client name is
  # chef-validator, unless you changed the configuration.
  #
  #   chef.validation_client_name = "ORGNAME-validator"
end
