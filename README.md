# centos49_gcc482

## Memo

- `/opt/tools346` : built w/ gcc-3.4.6 environment
- `/opt/tools482` : built w/ gcc-4.8.2 environment
- `/tools482 => /opt/tools482` : Sym link

## Environment
- Host: Windows 10, Ubuntu 16.04
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) 5.1.6 + ExtensionPack
- [Vagrant](https://www.vagrantup.com/downloads.html) 1.8.6
- vagrant box : [mt08/centos49-i386](https://vagrantcloud.com/mt08/boxes/centos49-i386)


## Instructions
### Prepare VM environment
1. Host(Win): Install [git for windows](https://git-scm.com/download)
2. Host: Install Vagrant
3. Host: Intall VirtualBox
4. Host(Win?): Reboot
5. Host: Intall VirtualBox ExtensionPack
6. Host: Install vagrant plugin :: open bash `vagrant plugin install vagrant-vbguest`

### Start VM on Windows
1. Open `git-bash` with Administrator privileges
2. Makw work folder and vagrant init

    ```bash
mkdir -p /c/vagrant/centos49
cd /c/vagrant/centos49
vagrant init mt08/centos49-i386 --minimal
```
3. Edit `Vagrantfile` for port-fowarding and system resources (num of CPUs, Memory, ...)

    ```bash
$ cat Vagrantfile
    Vagrant.configure("2") do |config|
      config.vm.box = "mt08/centos49-i386"
      config.vm.network "forwarded_port", guest: 80, host: 30080
      config.vm.network "forwarded_port", guest: 443, host: 30443
      config.vm.network "forwarded_port", guest: 3000, host: 33000
      config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "4096", "--cpus", "2", "--ioapic", "on"]
      end
end
```

4. `vagrant up`
    - For the first time, it may take time to download ~400MB box file. 
    
5. After VM has started, you'll see the message `"default: Warning: Authentication failure. Retrying..."`  continuously.

    => Open another `git-bash` window w/ Admin-priv, and `vagrant ssh` to login with password `vagrant`, and type:
    ```bash
 chmod 600 ~/.ssh/authorized_keys
 ```

6. Then it will continue to install VirtualBox GuestAdditions.

    => After this, Guest `/vagrant` folder is mounted to the current work folder on Host. (You'll see `Vagrantfile`.) 

#### Memo
- Turn on VM: `vagrant up`
- Login to VM: `vagrant ssh`
- Shutdown: `vagrant halt`
- Reboot: `vagrant reload`
- Delete: `vagrant destroy`



## Install gcc 4.8.2 Environment
1. Host: `git clone https://github.com/mt08xx/centos49_gcc482.git` to the same folder where 'Vagrantfile' is.

    ```
cd /c/vagrant/centos49
git clone https://github.com/mt08xx/centos49_gcc482.git
```

    => Then, you'll see the files under `/vagrant` on guest. ( `ls -l /vagrant/centos49_gcc482/binary` )

2. Guest: copy and paste ...

    ```bash
#
sudo mkdir -p /opt/tools482 /opt/tools346 /var/log/porg
sudo chmod 777 -R /opt/tools482 /opt/tools346 /var/log/porg
sudo chown vagrant.vagrant /opt/tools482 /opt/tools346 /var/log/porg
sudo ln -svf  /opt/tools482 /
#
PATH=/opt/tools482/bin:/opt/tools346/bin:$PATH
grep /opt/tools482 .bashrc || echo 'PATH=/opt/tools482/bin:/opt/tools346/bin:$PATH' >> $HOME/.bashrc
#
tar zxvf /vagrant/centos49_gcc482/binary/porg-0.10.porg.tar.gz -C /
tar zxvf /vagrant/centos49_gcc482/binary/tar-1.29.porg.tar.gz -C /
hash -r
#
#
# Copy files
mkdir -p /tools482/include/
cp -rv /usr/include/* /tools482/include/
#
#
# Extract files
cd $HOME
DIR_WORK_PORGBALL=$HOME/work_porg
for f in xz-5.2.2 python-2.7.12 make-4.2.1 binutils-2.24-pass2  glibc-2.11.3 libstdc++-4.8.2 gcc-4.8.2-pass2 ; do echo $f
    rm -rfv $DIR_WORK_PORGBALL
    mkdir $DIR_WORK_PORGBALL
    porgball -e --root=$DIR_WORK_PORGBALL /vagrant/centos49_gcc482/binary/$f.porg.tar.*
    porg -lp $f "cp -r $DIR_WORK_PORGBALL/* /"
    rm -rf $DIR_WORK_PORGBALL
    hash -r
done
```


3. `gcc -v`

    ```
[vagrant@localhost ~]$ gcc --version
gcc (GCC) 4.8.2
Copyright (C) 2013 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

[vagrant@localhost ~]$ 
```
