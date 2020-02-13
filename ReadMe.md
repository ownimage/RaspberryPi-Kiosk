# Setting up the Raspberry Pi
First we need to get openssh-server working.
##On the Raspberry Pi
I have started with the Ubuntu 18.04 Mate distribution.
```
sudo apt-get remove --purge openssh-server
sudo apt-get install openssh-server
sudo service ssh status

sudo cp  /etc/ssh/sshd_config  /etc/ssh/sshd_config.original
sudo chmod +w  /etc/ssh/sshd_config
sudo vi  /etc/ssh/sshd_config
```

Add the following line under the `PermitRootLogin` commented out line
```
PermitRootLogin yes
```

And also remove the comment from 
```
PasswordAuthentication yes
```

Then
```
sudo service ssh restart
sudo passwd <password>
```
This should be sufficient that on the `client machine` you can do:
```
ssh root@<ip address>
```

And it should allow you to enter the password and login

##On the client machine
Now we want to change things so that we can log in as root without having to give the password.
Note in the following `rpi` is just the name of an identity ... it could be something replaced with something different.

```
ssh-keygen -f rpi
mv rpi rpi.pub  ~/.ssh
ssh-copy-id -i ~/.ssh/rpi root@<ipaddress>
```

Test that the key has worked
```
ssh -i ~/.ssh/rpi root@<ipaddress>
```
Or to do this without needing to specify the `-i` parameter set up `ssh-agent` as follows.
```
eval `ssh-agent`
ssh-add <path-to-users_home-dir>/.ssh/rpi
ssh root@<ipaddress>
```
This should let up login in without needing to give the password.  The `eval` and `ssh-add` commands will need to be run in each session.
If that works we can now disable root login with a password on the Raspberry Pi.

##On the Raspberry Pi
```
sudo passwd -l root
sudo vi  /etc/ssh/sshd_config
```
Change the following :
```
#PasswordAuthentication yes
PermitRootLogin yes
```
We now need to restart the server to make this take effect.
```
sudo service ssh restart
```
##On the client machine
```
ssh -i ~/.ssh/rpi root@<ipaddress>
```
Should let you login.

Then to save having to use the -i each time the following should work for your session:
```
eval `ssh-agent`
ssh-add <path-to-users_home-dir>/.ssh/rpi
ssh root@<ip address> 
```

#Running Ansible
You will need to change the ip address in hosts.ini to be the same as the Pi that you are setting up.
```
ansible-playbook -i hosts.ini  -u root install-kiosk.yml
```