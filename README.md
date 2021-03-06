# saltstack-lxc-vagrant
Vagrantfile for spinning up a local Salt Master along with as many minions as
you need defined in a YAML file using LXC containers.

At present this vagrantfile only supports Ubuntu boxes due to an Ubuntu
specific Salt bootstrap used by the minions. I'm planing to make use of the
official salt bootstrap in the future to better support other OSs (please feel
free to make pull requests to help with this!)

## Requirements

This assumes Ubuntu a 14.04 host, package names are likely vary in other distros.

Packages:

1. [vagrant 1.6+](http://www.vagrantup.com/downloads.html)
2. lxc 0.7.5+
3. cgroups-lite
4. redir
5. [kernel !=3.5.0-17.28](https://github.com/fgrehm/vagrant-lxc/wiki/Troubleshooting#im-unable-to-restart-containers)

Vagrant plugins:

1. [vagrant-lxc](https://github.com/fgrehm/vagrant-lxc)
2. [salty-vagrant-grains](https://github.com/ahmadsherif/salty-vagrant-grains)

### Environmental variables

Not required but it makes your life easier if you use this a a lot.

    VAGRANT_DEFAULT_PROVIDER=lxc
    VAGRANT_DOTFILE_PATH=~/.vagrant

## Salt Master

To use the Salt Master (and be able to provisions systems) you'll need to clone
your Salt Stack repos into the following directories:

    git clone git:your-states-repo ~/git/saltstack
    git clone git:your-pillar-repo ~/git/saltstack-pillar

The master will then use whichever states/pillars you have checked out locally
which means you can edit your states in the comfort of your own environment
while having your development machines  isolated and easily replaceable.

The Salt Master runs the `fgrehm/trusty64-lxc` box on the IP `10.0.3.2`.

## Minions

### (easily) Available Boxes

As stated above, only Ubuntu is supported atm so the box choice is a little
limited.

| Distribution | VagrantCloud box |
| ------------ | ---------------- |
| Ubuntu Precise 12.04 x86_64 | [fgrehm/precise64-lxc](https://vagrantcloud.com/fgrehm/precise64-lxc) |
| Ubuntu Trusty 14.04 x86_64 | [fgrehm/trusty64-lxc](https://vagrantcloud.com/fgrehm/trusty64-lxc) |


### Define minion
To define minions you need to create a `minions.yaml` in the repo directory
with the following format:

    - name: minion-box
      box: fgrehm/trusty64-lxc
      ram: 512M
      ip: 10.0.3.11

### Shared folders
If you need to share folders with the minion you can add them in like so:

    - name: minion-box
      box: fgrehm/trusty64-lxc
      ram: 512M
      ip: 10.0.3.11
      folders:
        - from: ~/source/folder
          to: /destination/folder

You can repeat this as many times as needed:

      folders:
        - from: ~/source/folder1
          to: /destination/folder1
        - from: ~/source/folder2
          to: /destination/folder2
        - from: ~/source/folder3
          to: /destination/folder3

### Salt grains
There are some default grains that can't be overwritten with the YAML file, if
you need to change these you'll need to change them in the vagrant file itself.

The reason for this is because I use conditionals in a few states (things like
updating DNS via a script) and it prevents any accidental triggering of these
states.

      grains:
          environment: development
          provider: vagrant

If you need to define some grains so you can do so as below:

      grains:
        key: value
        list:
          - item1
          - item2

### Example minion.yaml

    - name: minion-box
      box: fgrehm/precise64-lxc
      ram: 512M
      ip: 10.0.3.11
      folders:
        - from: ~/git/my-awesome-website
          to: /var/www/my-awesome-website
      grains:
        role: webserver
        list:
          - item1
          - item2

# Time to play!
Once you've setup the minions.yaml you can check everything out.

````vagrant status````

First we need to fire up the Salt Master.

````vagrant up saltmaster````

Now you can fire up your minions as needed.

````vagrant up minion-box````

By default this vagrantfile does not run a highstate as I find it can slow
things down when developing states.

Once it's all up and running you can ssh into it and run a highstate manually

````vagrant ssh minion-box````
````sudo salt-call state.highstate````

# Notes

I do not suggest firing up both master and minions at the same time as this can
lead to a broken setup where the minions are up before the master which means
you need to manually restart the salt-minion on each minion box to get them
taliking to the master properly, and that's no fun.
