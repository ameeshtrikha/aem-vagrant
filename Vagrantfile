# -*- mode: ruby -*-
# vi: set ft=ruby :

require "yaml"


Vagrant.require_version ">= 1.7.4"

# Method to check if a local port it open
def is_local_port_open?(port)
  begin
    Timeout::timeout(2) do
      begin
        s = TCPSocket.new('127.0.0.1', port)
        s.close
        return true
      rescue Errno::ECONNREFUSED, Errno::EHOSTUNREACH
      end
    end
  rescue Timeout::Error
  end
  return false
end

# Method to configure the box name and url (if exists)
def configure_box(config, box_name, box_url, vm_name)
  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = box_name

  if box_url
    # The url from where the 'config.vm.box' box will be fetched if it
    # doesn't already exist on the user's system.
    config.vm.box_url = box_url
  end
end

# Method to confiure port forwards http://static.content.akqa.net/static/content/jre/ 
def configure_port_forwarding(config, port_forwards)
  port_forwards.each do |name, forward|
  	if is_local_port_open? forward['host_port']
        puts "[WARN] Port #{forward['host_port']} is in use."
    end
  	config.vm.network :forwarded_port, guest: forward['guest_port'], host: forward['host_port']
  end
end

# Method to configure share folders
def configure_shared_folders(config, shared_folders)
  shared_folders.each do |name, folder|
    config.vm.synced_folder folder['host_path'], folder['vm_path'], :mount_options => ['dmode=777,fmode=666'], :create => true
  end
end

# Method to configure the provider
def configure_provider(config, vm_name, cpus, memory_size, box_os)
  config.vm.provider :virtualbox do |vb|
    # Configure VM resources
    vb.gui = false
    vb.name = vm_name
    vb.cpus = cpus
    vb.memory = memory_size
  
    # allow symlinks to creation in default vagrant share folder https://github.com/mitchellh/vagrant/issues/713
    vb.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
    if box_os == "ubuntu"
      # Fix for network name lookups broken in NAT network adapters
      # see https://bugs.launchpad.net/ubuntu/+source/virtualbox/+bug/1048783
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
  end
end

# Method to configure private network IP
def configure_private_network_ip(config, ip, vm_name)
  if ip
    config.vm.network :private_network, :ip => ip, :netmask => "255.255.255.0"
  else
    puts "       NO HOSTONLY IP defined for VM #{vm_name}."
  end
end

# Method to configure host name
def configure_host_name(config, host_name, vm_name)
  if host_name
    config.hostsupdater.remove_on_suspend = true
    config.vm.host_name = host_name
    config.hostsupdater.aliases = [ "secure.#{host_name}" ]
  end
end

# Method to Load default config from config.yaml. If config_local.yaml exists then merge into default config.
def deep_merge!(src, override)
  override.each do |key, oval|
    # puts "merging #{key}"
    if src.has_key?(key)
      sval = src[key]
      if oval == sval
        # puts "Values are identical"
        next
      elsif oval.is_a?(Hash) and sval.is_a?(Hash)
        # puts "Deep merging subhashes"
        deep_merge!(sval, oval)
      elsif oval.is_a?(Array) and sval.is_a?(Array)
        # puts "Deep merging arrays"
        sval.concat oval
      else
        # puts "Overriding value #{sval.inspect} with #{oval.inspect}"
        src[key] = oval
      end
    else
      # puts "adding new value {#{key} => #{oval}}"
      src[key] = oval
    end
  end
  return src
end

# Load default config from config.yaml. If config_local.yaml exists then merge into default config.
# By default keep all VMs disabled in config.yaml and enable specific VMs in config_local.yaml.
CONF = YAML.load(IO.read(File.join(File.dirname(__FILE__), "config.yaml")))
begin
  deep_merge!(CONF, YAML.load(IO.read(File.join(File.dirname(__FILE__), "config_local.yaml"))))
# Throw no such file error.
rescue Errno::ENOENT
end


def configure_chef_solo_provision(config, vm_conf)
    config.vm.provision :chef_solo do |chef|

    chef.log_level = :info
    
    if vm_conf['box_os'] == "ubuntu"
      chef.add_recipe "apt"
    elsif vm_conf['box_os'] == "centos"
      chef.add_recipe "yum"
    end
    chef.add_recipe "nano"
    chef.add_recipe "java"
    chef.add_recipe "aem::aem_setup"
    if vm_conf['install_dispatcher']
      chef.add_recipe "dispatcher-simplified-any"
    end 

    chef.json = {
        :java => {
            :install_flavor => "oracle",
            :jdk => {
                vm_conf['jdk_version'] => {
                    :x86_64 => {
                        :url => vm_conf['java']['x86_64']['url'],
                        :checksum => vm_conf['java']['x86_64']['checksum']
                    },
                    :i586 => {
                        :url => vm_conf['java']['i586']['url'],
                        :checksum => vm_conf['java']['i586']['checksum']
                    }
                }
            },
            :jdk_version => vm_conf['jdk_version'],
            :oracle => {
                :accept_oracle_download_terms => true
            },
        },
        :aem => {
            :is_reinstall => false,
            :version => "6_x",
            :install_packages => vm_conf['install_packages'],
            :installation_type => vm_conf['installation_type'],
            :sling_run_modes => vm_conf['sling_run_modes'],
            :port => vm_conf['port'],
            :min_heap => vm_conf['min_heap'],
            :max_heap => vm_conf['max_heap'],
            :jvm => {
              :options => vm_conf['jvm_options']
            },
            :license_url => vm_conf['license_url'],
            :"deploy" => {
                :jar_url => vm_conf['jar_url']
            }
        }
    }
  end
end

# configure VM'S 
Vagrant.configure(2) do |global_config|

  # Enable berkshelf support
    global_config.berkshelf.enabled = true
    global_config.berkshelf.berksfile_path = "Berksfile"

    CONF['vms'].each do |vm_name, vm_conf|
        if vm_conf['enabled']
            global_config.vm.define vm_name do |config|
            	  # Enable berkshelf support
  				      config.berkshelf.enabled = true

  				      # Use Omnibus to install the latest Chef
  				      config.omnibus.chef_version = :latest

                # Configure the Box Name and URL (if exists)
                configure_box(config, vm_conf['box_name'], vm_conf['box_url'], vm_name)

                # Boot with a GUI so you can see the screen. (Default is headless)
                config.vm.boot_mode = :gui if vm_conf['gui']
				
				        # Configure Port Forwarding
				        configure_port_forwarding(config, vm_conf['port_forwards'])

                # Configure Shared Folders
                configure_shared_folders(config, vm_conf['shared_folders'])
               	
                # VirtualBox Specific Configuration
                configure_provider(config, vm_name, vm_conf['cpus'], vm_conf['memory_size'], vm_conf['box_os'])
   				      
                # Setup Private Network IP & Bridge
                configure_private_network_ip(config, vm_conf['ip'], vm_name)

                # Setup Guest HostName
                configure_host_name(config, vm_conf['host_name'], vm_name)

                # Configure chef_solo
                configure_chef_solo_provision(config, vm_conf)
            end
        end
    end
end
