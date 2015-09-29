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

A simple VM definition:
```ruby
  # Linux machine
  c.vm.define "linacs-1" do |config|
    config.vm.box           = 'Centos-template-name'
  end
```



