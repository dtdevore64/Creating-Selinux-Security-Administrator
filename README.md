# **Creating-Selinux-Security-Adminstrator Role**
<br><br>

What is a Security Adminstrator Role in Selinux? A Secruity Adminstrator Role in Selinux is someone who can adminstrate the policy. For example they can control whether the policy is in enforcing or permissive mode. Usually the System Adminstrator would just control the policy but if you want to implement RBAC(Role Based Access Control) you would create a role specifically just to implement the policy and that's all they would do. In this example I am going to install Selinux from scratch on a new Fedora system and create a new Linux user and add the Security Adminstrator role to said user and then finally map them together.
<br><br>



***Step 1.*** Install Selinux on Fedora 34

```
sudo dnf install selinux*
```
<br><br><br><br>


***Step 2.*** Install more stuff related to Selinux that we will need
```
sudo dnf install secilc* -y
sudo dnf install setroubleshoot* -y
sudo dnf install policycoreutils* -y
sudo dnf install setools* -y

```
<br><br><br><br>

***Step 3.*** Create new Linux user and map it to an Selinux user
```
   sudo useradd joe
   sudo passwd joe
   sudo semanage login a -s staff_u -r s0-s0:c0.c1023 joe
```
<br><br><br><br>

***Step 4.*** Adding the security adminstrator role to the staff_u selinux user
```
sudo semanage user -m -R "secadm_r staff_r sysadm_r system_r unconfined_r" staff_u
```
<br><br><br><br>

***Step 5.*** Adding the Security Adminstrator Role to be able to use sudo in the ```"/etc/sudoers"``` file
```
##  Allow root to run any commands anywhere
root  ALL=(ALL)   ALL
joe   ALL=(ALL) ROLE=secadm_r TYPE=secadm_t ALL
```

<br><br><br><br>

***Step 6.*** Now we will be using the restorecon command to fix the labels in our home directory and then reboot afer that
```
sudo restorecon -RF -v /home/joe
sudo shutdown -r now
```
<br><br><br><br>


***Step 7.*** We must disable the selinux policy package(.pp) allowing the sysadm adminstrator to mess with the policy since we only want our secadm to control the policy and no one else. To do that we will disable the ```"sysadm_secadm.pp"```
```
sudo semodule -d sysadm_secadm -X 100
```

Also if you would like to find that specific policy package and the rest of the packages just run the following commands: ```sudo semodule -lfull | grep sysadm_secadm``` and ```sudo semodule -lfull```
<br><br><br><br>

***Step 8.*** Now whenever we log in we will always be in the staff_u user, staff_r role, and the staff_t type as the default. So now whenever we use the command ```"sudo"``` it will transition us from the staff_r and staff_t types to the secadm_r and secadm_t types. To test this just run the following command:

```
sudo id -Z
```

