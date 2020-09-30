# Drupal VM (Alpha Variant) Quick Start

Drupal VM - Alpha is a set of small alterations to the amazing core built by geerlingguy. It's focus is streamlining the multiple sites in one environment use-case as well as add additional items by default to aid front end development.

## Software Prerequisites
* XCode - OS X Command Line Tools (https://itunes.apple.com/us/app/xcode/id497799835?mt=12)
 * Run `xcode-select --install` in terminal after install
* Virtual Box (https://www.virtualbox.org/)
* Vagrant (https://www.vagrantup.com/)

## Quick Start (Mac)
1. Install PIP: `sudo easy_install pip`
2. Install Ansible: `sudo pip install ansible`
3. Clone this repository: `git clone https://github.com/akeemw/drupal-vm.git`
4. In Terminal.app `cd` into the "drupal-vm" folder.
5. Copy local.config.sample.yml and create local.config.yml
6. Add projects to local.config.yml (more information below)
7. Run `vagrant up`

## How do I add a project?
Add a new item into "apache_vhosts" in the local.config.yml file with "is_project" set to true.

      - name: "myproject"
        servername: "myproject.dev"
        documentroot: "/var/www/myproject"
        serveralias: "myproject.*.xip.io"
        is_project: true
        gulp_root: "/var/www/myproject/themes/mytheme"
        extra_parameters: "{{ apache_vhost_php_fpm_parameters }}"

The "gulp_root" setting is optional and used only for tmuxinator.

## Where can I find my projects?
The `~/Sites/drupalvm-alpha` folder on your Mac will be mounted within the Vagrant VM at `/var/www/`.

## Can I run a shorter provision process to set up a new site?

Yes, run the following line in terminal:

      DRUPALVM_ANSIBLE_TAGS=projects vagrant provision

## Tmux & Tmuxinator

[Tmux](https://github.com/tmux/tmux) & [Tmuxinator](https://github.com/tmuxinator/tmuxinator) are installed by default and tmuxinator configuration files will be created for all apache vhosts flagged with "is_project: true".

By default each session is configured to have windows for:
* Project Document Root
* Tail of the Apache error log
* Root for Gulp execution (requires "gulp_root" option to be specified on the project)

Configuration files are created in `~/.config/tmuxinator/`

Launch a Project Session:

      tmuxinator projectname

Switch Between windows with the following key:

1. Do the following key combination: Ctrl-b
2. Type Window Number

Leave a Project Session:

      tmux detach

Close a Project Session:

      tmux kill-session -t projectname

## Port forwards
Port forwarding has been configured to allow HTTP access into the Vagrant box by
other devices on the same network (e.g. a smartphone for testing purposes).

| Host Port | Guest Port | Note     |
|-----------|------------|-----------
| 9999      | 80         | External access (e.g. projectname.127.0.0.1.xip.io:9999) |
| 3333      | 3000       | [BrowserSync](http://www.browsersync.io) (e.g. projectname.127.0.0.1.xip.io:3333) |

## Enabling SSH Agent forwarding (Mac)
SSH Agent forwarding is required if you want to be able to use your Mac's SSH keys from within the VM.

1. Add the following to `~/.ssh/config`

        Host 192.168.88.88
           ForwardAgent yes

2. In Terminal.app add Keys to the SSH Agent. For example:

        /usr/bin/ssh-add -K ~/.ssh/id_rsa.pub

## Changes & Additional Features
* Port forwarding for ports 80 and 3000 (browsersync).
* Addition of a 'dev-box-alpa' example config file.
* Addition of new projects task
* Addition of slim, projects-only provision
* Tmux & tmuxinator installed by default. All projects have a tmuxinator configuration.

---

![Drupal VM Logo](https://raw.githubusercontent.com/geerlingguy/drupal-vm/master/docs/images/drupal-vm-logo.png)

[![Build Status](https://travis-ci.org/geerlingguy/drupal-vm.svg?branch=master)](https://travis-ci.org/geerlingguy/drupal-vm) [![Documentation Status](https://readthedocs.org/projects/drupal-vm/badge/?version=latest)](http://docs.drupalvm.com) [![Packagist](https://img.shields.io/packagist/v/geerlingguy/drupal-vm.svg)](https://packagist.org/packages/geerlingguy/drupal-vm) [![Docker Automated build](https://img.shields.io/docker/automated/geerlingguy/drupal-vm.svg?maxAge=2592000)](https://hub.docker.com/r/geerlingguy/drupal-vm/) [![](https://images.microbadger.com/badges/image/geerlingguy/drupal-vm.svg)](https://microbadger.com/images/geerlingguy/drupal-vm "Get your own image badge on microbadger.com") [![irc://irc.freenode.net/drupal-vm](https://img.shields.io/badge/irc.freenode.net-%23drupal--vm-brightgreen.svg)](https://riot.im/app/#/room/#drupal-vm:matrix.org)

[Drupal VM](https://www.drupalvm.com/) is a VM for Drupal, built with Ansible.

Drupal VM makes building Drupal development environments quick and easy, and introduces developers to the wonderful world of Drupal development on virtual machines or Docker containers (instead of crufty old MAMP/WAMP-based development).

There are two ways you can use Drupal VM:

  1. With Vagrant and VirtualBox.
  2. With Docker.

The rest of this README assumes you're using Vagrant and VirtualBox (this is currently the most flexible and widely-used method of using Drupal VM). If you'd like to use Drupal VM with Docker, please read the [Drupal VM Docker documentation](http://docs.drupalvm.com/en/latest/other/docker/).

Drupal VM installs the following on an Ubuntu 18.04 (by default) linux VM:

  - Apache (or Nginx)
  - PHP (configurable version)
  - MySQL (or MariaDB, or PostgreSQL)
  - Drupal 7, 8, or 9
  - Optional:
    - Drupal Console
    - Drush
    - Varnish
    - Apache Solr
    - Elasticsearch
    - Node.js
    - Selenium, for testing your sites via Behat
    - Ruby
    - Memcached
    - Redis
    - SQLite
    - Blackfire, XHProf, or Tideways for profiling your code
    - XDebug, for debugging your code
    - Adminer, for accessing databases directly
    - Pimp my Log, for easy viewing of log files
    - MailHog, for catching and debugging email

It should take 5-10 minutes to build or rebuild the VM from scratch on a decent broadband connection.

Please read through the rest of this README and the [Drupal VM documentation](http://docs.drupalvm.com/) for help getting Drupal VM configured and integrated with your workflow.

## Documentation

Full Drupal VM documentation is available at http://docs.drupalvm.com/

## Customizing the VM

There are a couple places where you can customize the VM for your needs:

  - `config.yml`: Override any of the default VM configuration from `default.config.yml`; customize almost any aspect of any software installed in the VM (more about [configuring Drupal VM](http://docs.drupalvm.com/en/latest/getting-started/configure-drupalvm/).
  - `drupal.composer.json` or `drupal.make.yml`: Contains configuration for the Drupal core version, modules, and patches that will be downloaded on Drupal's initial installation (you can build using Composer, Drush make, or your own codebase).

If you want to use Drupal 8 on the initial install, do the following:

  1. Set `drupal_major_version: 8` inside `config.yml`.
  2. Set `drupal_composer_project_package: "drupal/recommended-project:^8@dev"` inside `config.yml`.
  
  
If you want to use Drupal 7 on the initial install, do the following:

  1. Switch to using a [Drush Make file](http://docs.drupalvm.com/en/latest/deployment/drush-make/).
  1. Update the Drupal `version` and `core` inside your `drupal.make.yml` file.
  2. Set `drupal_major_version: 7` inside `config.yml`.

## Quick Start Guide

This Quick Start Guide will help you quickly build a Drupal 9 site on the Drupal VM creating a new Composer project. You can also use Drupal VM with [Composer](http://docs.drupalvm.com/en/latest/deployment/composer/), a [Drush Make file](http://docs.drupalvm.com/en/latest/deployment/drush-make/), with a [Local Drupal codebase](http://docs.drupalvm.com/en/latest/deployment/local-codebase/), or even a [Drupal multisite installation](http://docs.drupalvm.com/en/latest/deployment/multisite/).

If you want to install a Drupal site locally with minimal fuss, just:

  1. Install [Vagrant](https://www.vagrantup.com/downloads.html) and [VirtualBox](https://www.virtualbox.org/wiki/Downloads).
  2. Download or clone this project to your workstation.
  3. `cd` into this project directory and run `vagrant up`.

But Drupal VM allows you to build your site exactly how you like, using whatever tools you need, with almost infinite flexibility and customization!

### 1 - Install Vagrant and VirtualBox

Download and install [Vagrant](https://www.vagrantup.com/downloads.html) and [VirtualBox](https://www.virtualbox.org/wiki/Downloads).

You can also use an alternative provider like Parallels or VMware. (Parallels Desktop 11+ requires the "Pro" or "Business" edition and the [Parallels Provider](http://parallels.github.io/vagrant-parallels/), and VMware requires the paid [Vagrant VMware integration plugin](http://www.vagrantup.com/vmware)).

Notes:

  - **For faster provisioning** (macOS/Linux only): *[Install Ansible](http://docs.ansible.com/intro_installation.html) on your host machine, so Drupal VM can run the provisioning steps locally instead of inside the VM.*
  - **For stability**: Because every version of VirtualBox introduces changes to networking, for the best stability, you should install Vagrant's `vbguest` plugin: `vagrant plugin install vagrant-vbguest`.
  - **NFS on Linux**: *If NFS is not already installed on your host, you will need to install it to use the default NFS synced folder configuration. See [nfs instructions for Linux](http://docs.drupalvm.com/en/latest/getting-started/installation-linux/#mounting-nfs-shared-folders-hangs)*
  - **Versions**: *Make sure you're running the latest releases of Vagrant, VirtualBox, and Ansible—as of 2020, Drupal VM recommends: Vagrant 2.2.x, VirtualBox 6.1.x, and Ansible 2.9.x*

### 2 - Build the Virtual Machine

  1. Download this project and put it wherever you want.
  2. (Optional) Copy `default.config.yml` to `config.yml` and modify it to your liking.
  3. Create a local directory where Drupal will be installed and configure the path to that directory in `config.yml` (`local_path`, inside `vagrant_synced_folders`).
  4. Open Terminal, `cd` to this directory (containing the `Vagrantfile` and this README file).
  5. Type in `vagrant up`, and let Vagrant do its magic.

Once the process is complete, you will have a Drupal codebase available inside the `drupal/` directory of the project.

Note: *If there are any errors during the course of running `vagrant up`, and it drops you back to your command prompt, just run `vagrant provision` to continue building the VM from where you left off. If there are still errors after doing this a few times, post an issue to this project's issue queue on GitHub with the error.*

### 3 - Access the VM.

Open your browser and access [http://drupalvm.test/](http://drupalvm.test/). The default login for the admin account is `admin` for both the username and password.

Note: *By default Drupal VM is configured to use `192.168.88.88` as its IP, if you're running multiple VM's the `auto_network` plugin (`vagrant plugin install vagrant-auto_network`) can help with IP address management if you set `vagrant_ip` to `0.0.0.0` inside `config.yml`.*

## Extra software/utilities

By default, this VM includes the extras listed in the `config.yml` option `installed_extras`:

    installed_extras:
      - adminer
      # - blackfire
      # - drupalconsole
      - drush
      # - elasticsearch
      # - java
      - mailhog
      # - memcached
      # - newrelic
      # - nodejs
      - pimpmylog
      # - redis
      # - ruby
      # - selenium
      # - solr
      # - tideways
      # - upload-progress
      - varnish
      # - xdebug
      # - xhprof

If you don't want or need one or more of these extras, just delete them or comment them from the list. This is helpful if you want to reduce PHP memory usage or otherwise conserve system resources.

## Using Drupal VM

Drupal VM is built to integrate with every developer's workflow. Many guides for using Drupal VM for common development tasks are available on the [Drupal VM documentation site](http://docs.drupalvm.com).

## Updating Drupal VM

Drupal VM follows semantic versioning, which means your configuration should continue working (potentially with very minor modifications) throughout a major release cycle. Here is the process to follow when updating Drupal VM between minor releases:

  1. Read through the [release notes](https://github.com/geerlingguy/drupal-vm/releases) and add/modify `config.yml` variables mentioned therein.
  2. Do a diff of your `config.yml` with the updated `default.config.yml` (e.g. `curl https://raw.githubusercontent.com/geerlingguy/drupal-vm/master/default.config.yml | git diff --no-index config.yml -`).
  3. Run `vagrant provision` to provision the VM, incorporating all the latest changes.

For major version upgrades (e.g. 4.x.x to 5.x.x), it may be simpler to destroy the VM (`vagrant destroy`) then build a fresh new VM (`vagrant up`) using the new version of Drupal VM.

## System Requirements

Drupal VM runs on almost any modern computer that can run VirtualBox and Vagrant, however for the best out-of-the-box experience, it's recommended you have a computer with at least:

  - Intel Core processor with VT-x enabled
  - At least 4 GB RAM (higher is better)
  - An SSD (for greater speed with synced folders)

## Other Notes

  - To shut down the virtual machine, enter `vagrant halt` in the Terminal in the same folder that has the `Vagrantfile`. To destroy it completely (if you want to save a little disk space, or want to rebuild it from scratch with `vagrant up` again), type in `vagrant destroy`.
  - To log into the virtual machine, enter `vagrant ssh`. You can also get the machine's SSH connection details with `vagrant ssh-config`.
  - When you rebuild the VM (e.g. `vagrant destroy` and then another `vagrant up`), make sure you clear out the contents of the `drupal` folder on your host machine, or Drupal will return some errors when the VM is rebuilt (it won't reinstall Drupal cleanly).
  - You can change the installed version of Drupal or drush, or any other configuration options, by editing the variables within `config.yml`.
  - Find out more about local development with Vagrant + VirtualBox + Ansible in this presentation: [Local Development Environments - Vagrant, VirtualBox and Ansible](http://www.slideshare.net/geerlingguy/local-development-on-virtual-machines-vagrant-virtualbox-and-ansible).
  - Learn about how Ansible can accelerate your ability to innovate and manage your infrastructure by reading [Ansible for DevOps](http://www.ansiblefordevops.com/).

## Tests

To run basic integration tests using Docker:

  1. [Install Docker](https://docs.docker.com/engine/installation/).
  2. In this project directory, run: `composer run-tests`

> Note: If you're on a Mac, you need to use [Docker's Edge release](https://docs.docker.com/docker-for-mac/install/#download-docker-for-mac), at least until [this issue](https://github.com/docker/for-mac/issues/77) is resolved.

The project's automated tests are run via Travis CI, and the more comprehensive test suite covers multiple Linux distributions and many different Drupal VM use cases and deployment techniques.

## License

This project is licensed under the MIT open source license.

## About the Author

[Jeff Geerling](https://www.jeffgeerling.com/) created Drupal VM in 2014 for a more efficient Drupal site and core/contrib development workflow. This project is featured as an example in [Ansible for DevOps](https://www.ansiblefordevops.com/).
