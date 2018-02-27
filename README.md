# Vagrant for AEM

Please read this file all the way through. Yes, really. There's important stuff in here !

This Vagrantfile and associated files will help you get set up quickly to run an Adobe CQ / AEM instance - AEM 6.x

The default Geometrixx packages will not be installed.

## Pre-Requisite
* Create folder structure /setup/aem/6.x
* Copy AEM_6.x_Quickstart.jar [name must match] to the folder __6.x__
* Copy license.properties [name must match] to the folder __6.x__

Currently the scripts are configured for 6.3, for any other version make changes accordingly.
## Installation

Install Vagrant 1.7.4 or later from the [Vagrant downloads page](http://www.vagrantup.com/downloads.html) and the latest [VirtualBox](https://www.virtualbox.org/)

*Verified to be working on Vagrant Version 1.7.4 and Virtualbox Version 5.0.14 r105127*

#### Install the Vagrant Berkshelf plugin.

This plugin requires the ChefDK, downloadable from here: http://downloads.getchef.com/chef-dk/

Windows: using '>= 2.0.1' will not be a recognized version on windows.  Using '2.0.1' will error. '3.0.1' works well. Using '4.0.0' will need the following fix: https://github.com/berkshelf/vagrant-berkshelf/issues/235 

    $ vagrant plugin install vagrant-berkshelf --plugin-version '>= 2.0.1'

#### Install the Vagrant Omnibus plugin

    $ vagrant plugin install vagrant-omnibus
    
#### Install the Vagrant Hostsupdater plugin

    $ vagrant plugin install vagrant-hostsupdater

## Usage

1. Create a new folder to hold the files.

2. Download *all* the files from the folder holding the type of instance you need. You should have `Vagrantfile`, `Berksfile`, `config.yaml` and `config_local_example.yaml`. 
3. `config.yaml` defines all the configuration parameters for AEM 6.x [author and publish] instances. 
4. Create a copy of `config_local_example.yaml` and name it as `config_local.yaml`
5. Edit the file `config_local.yaml` and set `enable` `true` for the instances you want to use. Please note you can either get AEM 6.0 instance type (author/publish) up or AEM 6.1 instance up, same instances for both can not be used simultaneously as it will cause port conflict in VirtualBox. 

6. Now from your shell run

        $ vagrant up

7. This will bring up the instances you enabled in `config_local.yaml`
8. You may be asked to enter you system password in order to allow the `host` entry for your machine in the system host file.
9. Also in future you wish to bring up only one of the instances, you could do -

        $ vagrant up <instance name>

	For example:  If you want to get AEM 6.1 author up, all you have to do is - 

        $ vagrant up aem6.1-author-vm
        
	**Its important to note that in order to use the _`instance name`_ , the instance name must be `enabled` `true` in config_local.yaml**

10. To ssh into an working instance you can do -
 
        $ vagrant ssh <instance name>


Note that once the Vagrant install has completed CQ is still installing itself, and so won't be instantly available.
    When it is installed then for the Author install you can use

    * [Author GUI](http://localhost:4502)

    For Publish you can use

    * [Direct, bypassing the dispatcher](http://localhost:4503)
    * [Through the dispatcher](http://localhost:8080)

## Where stuff lives on the guests

The CQ / AEM install lives under the `/data` directory, and the dispatcher writes under there for the Publish instance.
An init.d script is also installed under `/etc/init.d` in the usual Linux manner.


## Troubleshooting

### Network issues
The first time you start a new Vagrant installation be careful around using VPNs at the same time - if you disconnect or connect to the VPN after starting up vagrant it can cause issues with networking.

If you think this may have happened, disconnect from the VPN and do 

    $ vagrant destroy <instance name | optional> 
    $ vagrant up <instance name | optional>

This will restart the installation

### Updating software
Make sure you have the latest versions of [vagrant](http://www.vagrantup.com/), [virtualbox](https://www.virtualbox.org/) and the plugins. At the time of writing the
berkshelf plugin had just been updated for Vagrant 1.5 :

    $ rm -r ~/.vagrant.d/plugins.json ~/.vagrant.d/gems
    $ vagrant plugin install vagrant-berkshelf --plugin-version '>= 2.0.1'
    $ vagrant plugin install vagrant-omnibus
