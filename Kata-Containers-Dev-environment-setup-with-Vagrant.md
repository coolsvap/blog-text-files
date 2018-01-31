With reference to [Kata Containers Developers Guide](https://github.com/kata-containers/documentation/wiki/Developer-Guide) steps, I setted up the  development environment. At the same time, I went ahead and created a little automation to recreate the environment with [Vagrant](http://vagrantup.com/).

The primary code to create the environment is pushed at [vagrant-kata-dev](https://github.com/coolsvap/vagrant-kata-dev).

For setting it up, you will need,

- VirtualBox (Currently only tested with virtualbox)
- Vagrant with following plugins
  - vagrant-vbguest
  - vagrant-hostmanager
  - vagrant-share

To Install the plugins, use following command,

vagrant plugin install &lt;plugin-name&gt;

The setup instructions are simple, once you have installed the prereqs, clone the repo

git clone https://github.com/coolsvap/vagrant-kata-dev

Edit the Vagrantfile to update details

1. Update the bridge interface so the box will have IP address from your local network using DHCP. If you do not update, it will ask for the interface name you start machine.
2. Update the golang version, currently its at 1.9.3

Create the vagrant box with following command

vagrant up

Once the box is started, login to the box using following command

vagrant ssh

Switch to root user and move to vagrant shared directory and install the setup script

sudo su

cd /vagrant

./setup-kata-dev.sh

It will perform the steps required to setup the dev environment. Verify the setup done correctly with following steps,

# docker info | grep Runtime

WARNING: No swap limit support

Runtimes: kata-runtime runc

Default Runtime: runc
