Vagrant.configure("2") do |config|
  config.vm.provision "shell", privileged: true, inline: <<-EOS
    set -e -x

    apt-get -qq update
    apt-get install -y git ruby-dev build-essential libxml2-dev libxslt1-dev
    mkdir -p /usr/share/puppet/modules /usr/share/puppet/manifests /etc/puppet
    chown vagrant:vagrant /usr/share/puppet/modules
    touch /etc/puppet/hiera.yaml /usr/share/puppet/manifests/site.pp
  EOS

  # this is common to all boxes, but must be run *after* box-specific
  # provisioners. Vagrant's provisioner ordering rules say that the outer
  # scoped provisioners must run first, so that means each box must
  # individually invoke this script.
  install_puppet_code = <<-EOS
    set -e -x

    which bundler || sudo gem install bundler
    [ ! -e /usr/share/puppet/modules/phabricator ] && sudo ln -s /vagrant /usr/share/puppet/modules/phabricator
    (
      cd /vagrant
      bundle install
      bundle exec librarian-puppet install --path /usr/share/puppet/modules
    )
  EOS

  config.vm.network :forwarded_port, guest: 80, host: 8080

  config.vm.define "saucy" do |config|
    config.vm.box = "ubuntu-server-1310-x64-virtualbox"
    config.vm.box_url = "http://puppet-vagrant-boxes.puppetlabs.com/ubuntu-1310-x64-virtualbox-puppet.box"

    config.vm.provision "shell", privileged: false, inline: install_puppet_code
  end

  config.vm.define "precise" do |config|
    config.vm.box = "ubuntu-server-12042-x64-vbox4210"
    config.vm.box_url = "http://puppet-vagrant-boxes.puppetlabs.com/ubuntu-server-12042-x64-vbox4210.box"

    # puppetlabs says there are bugs in the precise ruby, so they include their
    # own ruby in /opt in their vagrant boxes. Configure the environment to
    # prefer those (the included puppet won't work otherwise)
    config.vm.provision "shell", privileged: true, inline: <<-EOS
      set -e -x

      PATH="/opt/ruby/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

      echo "Defaults secure_path=\"$PATH\"" > /etc/sudoers.d/use_vendor_ruby
      sudo chmod 0440 /etc/sudoers.d/use_vendor_ruby
      echo PATH='/opt/ruby/bin:$PATH' > /etc/profile.d/vagrantruby.sh
    EOS

    config.vm.provision "shell", privileged: false, inline: install_puppet_code
  end

  config.vm.define "trusty" do |config|
    config.vm.box = "ubuntu/trusty64"
    config.vm.provision "shell", privileged: false, inline: install_puppet_code
    config.vm.provider "virtualbox" do |v|
      v.memory = 1024
    end
  end
end
