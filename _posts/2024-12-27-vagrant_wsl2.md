---
layout: post
title:  "Hashicorp Vagrant in WSL2"
date:   2024-12-27 3:24:08 PM
categories: vagrant wsl wsl2 docker windows ubuntu
---
# Getting Vagrant 2.4.3 working in WSL2 with Ubuntu 22.04 LTS
As of Vagrant 2.4.3 I have had limited success with getting it working in a WSL2 environment.
I'm running Ubuntu 22.04 LTS in WSL2 and this post details the things I had to do to my vagrant
repo and the vagrant source code to get it working better in WSL2.

A lot of what you read on the net will be relevant to VirtualBox integration, however, I'm targeting the Docker
provider, not VirtualBox.

## Install Vagrant
[Install Instructions](https://developer.hashicorp.com/vagrant/downloads#linux)

## Notes on WSL2 and Windows Integration
The official Vagrant docs for [Vagrant and WSL Integration](https://developer.hashicorp.com/vagrant/docs/other/wsl)
state that if you have Vagrant installed in Windows, the version installed in WSL2 needs to match the one installed
in Windows. This is not relevant here as Vagrant is not installed in Windows.

Hashicorp also [recommend](https://developer.hashicorp.com/vagrant/docs/other/wsl#windows-access) exporting an
environment variable so that Hyper-V and VirtualBox providers are available to Vagrant from WSL2.
Since Docker will be used as the provider, this is not necessary.

You can omit this...

{% highlight shell %}
export VAGRANT_WSL_ENABLE_WINDOWS_ACCESS="1"
{% endhighlight %}

The other reason for exporting this environment variable is to access the Windows system and executables, and in
turn set the `VAGRANT_HOME` environment variable so that it is located in the Windows users home directory. This
is also not needed for our purposes.

## Changes Required

* **Problem**: Vagrant tries to share the vagrant project directory as a synced folder but is unable to share folders
  from WSL2. You will get an error stating that only DrvFs mounted directories can be shared.

  **Solution**: Have the Vagrant project directory also exist on a DrvFs mount and ensure your Vagrant project shares the
  location of the Vagrant directory located on the DrvFs folder. This also applies to any other folders you want to
  share. In my case Puppet is used for provisioning, so the `puppet` dir is also located on Windows and shared in
  the Vagrant project.

    ```ruby
        config.vm.define host.id, autostart: false do |h|
          h.vm.synced_folder "/mnt/c/dev/git/vagrant", "/vagrant"
          h.vm.synced_folder "/mnt/c/dev/git/puppet", "/home/vagrant/puppet"
    ```
    {% comment %}
    {% highlight ruby %}
      config.vm.define host.id, autostart: false do |h|
        h.vm.synced_folder "/mnt/c/dev/git/vagrant", "/vagrant"
        h.vm.synced_folder "/mnt/c/dev/git/puppet", "/home/vagrant/puppet"
    {% endhighlight %}
    {% endcomment %}

  To recap, duplicate copies of the Vagrant project will be located in both WSL2 and Windows,
  E.g. `/mnt/c/dev/git/vagrant` and you will run `vagrant` from WSL2, but your Vagrant project will refer to a
  synced folder on `/mnt/c/` (C:\\) to ensure the Vagrant project is shared from a DrvFs mounted directory.

* **Problem**: When sharing the DrvFs folders as synced folders to the Docker container, Vagrant will try to share
  them using a bind mount. It will execute `docker run -v <DrvFsDir>:<ContainerDir>` but within Vagrant source,
  the leading slash (/) is removed and thus the mount is treated as a volume mount, which fails
  [here in the Docker driver](https://github.com/hashicorp/vagrant/blob/v2.4.3/plugins/providers/docker/driver.rb#L103)
  I suspect it is line 94 causing the problem, but haven't verified.

  **Solution**: Patch the Vagrant source to fix this up.

    ```shell
      /opt/vagrant/embedded/gems/gems/vagrant-2.4.3/plugins/providers/docker$ diff driver.rb.orig driver.rb
102a103,106
      >           v.gsub!(/\\/, '/')
      >           v = '/' + v
      >           puts "Docker Provider: Volume: #{v.to_s}"
      >
    ```
    {% comment %}
    {% highlight shell %}
      /opt/vagrant/embedded/gems/gems/vagrant-2.4.3/plugins/providers/docker$ diff driver.rb.orig driver.rb
102a103,106
      >           v.gsub!(/\\/, '/')
      >           v = '/' + v
      >           puts "Docker Provider: Volume: #{v.to_s}"
      >
    {% endhighlight %}
    {% endcomment %}

