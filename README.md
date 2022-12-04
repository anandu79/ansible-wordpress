# Ansible-wordpress

Here, we will install WordPress in a remote server using Ansible.

For that, first we have to launch an amazon Linux instance from AWS and install Ansible in the server using the below command:

```
sudo amazon-linux-extras install ansible2 -y
```

Verify the installation by entering the below command:

```
ansible --version
```

Next, create a key file inside your working directory. 

```
vim aws.pem
```

Add your key in it and change the file permission:

```
chmod 400 aws.pem
```

Now, we will have to create a hosts file and add the remote instance which we prefer to install WordPress into it.

```
vim hosts
```

```
[your-group-name]    
Instance-IP-address ansible_user="ec2-user" ansible_port=22 ansible_ssh_private_key_file="aws.pem"
```

Check connectivity to the remote instance from the host instance by:

```
ansible -i hosts amazon -f 1 -m ping
```

Or

```
ansible -i hosts all -f 1 -m ping
```



























