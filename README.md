# vagrant-cloudstack-examples
Examples in using vagrant-cloudstack

## Step 0

### Step 0.1 Add your configuration information
Before any of the examples will work, put your personal configuration information for Cloudstack in yor environment.
As a working example, fill in the fields in CS_KEYS.sh and execute `. ./CS_KEYS.sh`
```bash
#!/bin/sh
export CLOUDSTACK_API_KEY=
export CLOUDSTACK_SECRET_KEY=
export CLOUDSTACK_HOST=
export PUBLIC_SOURCE_NAT_IP=
export NETWORK_NAME=
export SERVICE_OFFERING_NAME=
export ZONE_NAME=
```

### Step 0.2 Edit Vagrantfile(s, all!)
#### Step 0.2.1 for your ip/network
In the examples a connection will be allowed to the create VMs from a predefined network (`pf_trusted_networks`), make sure you change the Vagrantfile to your current external-visible IP (Look up at for example http://www.whatsmyip.org/)

#### Step 0.2.2 Edit your Vagrantfile to choose your VM template
The Vagrantfile specifies the template to be used in the section:
```ruby
config.vm.box           = 'Centos-template-name'
```
Replave the `Centos-template-name` with an apropriate template.

#### Step 0.2.3 Edit your Vagrantfile to specify your (template) credentials
The Vagrantfile specifies the credentials to use to connect (SSH) to the VM:
```ruby
    p.ssh_key               = './vagacs.key'
    p.keypair               = 'vagacs'
    p.ssh_user              = 'bootstrap'
```
* p.ssh_key: the private keyfile
* p.keypair: the name under which the key was created in Cloudstack
* p.ssh_user: the username of the 'bootstrap' user (e.g. 'root', 'vagrant')

### Step 0.3 Install Vagrant-Cloudstack plugin, or use bundler

On a working Vagrant installation, install the plugin 'vagrant-cloudstack':
```bash
vagrant plugin install vagrant-cloudstack
```

As an alternative you can work with latest development versions for plugin(s) or even Vagrant itself.
For this you need to create a Gemfile (or use/copy the provided Gemfile) in the respective (example) directory. Adjust it to you needs.
Execute in the example folder:
```bash
bundle install --path vendor/bundle --binstubs vendor/bin
```
NOTE to actually use the created bundle, pre-pend all vagrant commands with `bundle exec`, e.g. `bundle exec vagrant status`


## Example 0
If you have executed previous steps/edits, you should be able to do the following from within the example directories:

List (possibly) available machines:
```bash
$ # vagrant status
```
Running this will already verify your Cloudstack credentials and connection information.

Start a machine:
```bash
$ # vagrant up linacs-1
```

SSH into the VM:
```bash
$ # vagrant ssh linacs-1
```

Destroy the VM:
```bash
$ # vagrant destroy linacs-1
```

## Example 1

A simple Vagrantfile VM definition:
```ruby
  # Linux machine
  c.vm.define "linacs-1" do |config|
    config.vm.box           = 'Centos-template-name'
  end
```
## Example 2

A simple webserver installation using Chef (Vagrantfile):
```ruby
  c.omnibus.chef_version = '12.3.0'
  c.berkshelf.enabled = true
  
  # Linux machine
  c.vm.define 'linacs-1' do |config|
    config.vm.box           = 'Centos7-x86_64-Sbp_cis-XenServer-latest'
    config.vm.provision :chef_zero do |chef|
        chef.add_recipe 'apache2'
        chef.json = { :apache => { :default_site_enabled => true } }
    end
  end
```

In order for this example to work, the following installations are required:
* ChefDK: Download from https://downloads.chef.io/chef-dk/ , more instructions on https://docs.chef.io/install_dk.html
* To install chef-client in the VM; Vagrant plugin vagrant-omnibus: Execute `vagrant plugin install vagrant-omnibus`
* To collect the required cookbook(s); Vagrant plugin vagrant-berkshelf: Execute `vagrant plugin install vagrant-berkshelf`

To instruct vagrant-berkshelf to collect the 'apache2' cookbook, a 'Berskfile' has been created
```ruby
source "https://supermarket.chef.io"

#metadata

cookbook 'apache2'
```

## Example 3

Continuing on example 2, with portforwarding to test the webserver.

In the Vagrantfile the section c.vm.define extended:
```ruby
    config.vm.provision "shell", privileged: true, inline: 'echo Hello > /var/www/html/index.html'

    config.vm.provider :cloudstack do |p|
      p.port_forwarding_rules = [
        { :ipaddress => "#{ENV["PUBLIC_SOURCE_NAT_IP"]}", :protocol => "tcp", :publicport => 55550, :privateport  => 80, :openfirewall => false }
      ]
      p.firewall_rules = [
        { :ipaddress => "#{ENV["PUBLIC_SOURCE_NAT_IP"]}", :protocol => "tcp", :startport => 55550, :endport => 55550 }
      ]
    end
```

The `config.vm.provision "shell"` has been added to create an HTML file for Apache2 to publish

The `config.vm.provider :cloudstack do |p|` has been added at VM level to provide a portforward and firewall rule for this specific VM

If the run completed succesfully, it should provide a webpage, e.g. `wget -qO- http://85.222.237.60:55550`




