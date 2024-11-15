# Born2beroot

# Setting Up

## Installation Steps

1. **Setting Hostname**
   - Set the hostname as your login + "42" (e.g., `alexanfe42`).

2. **Domain Name**
   - Leave this field blank.

3. **Root Password**
   - Create and set a secure root password following the strong password policy.

4. **Full Name**
   - Leave this blank.

5. **Creating a Non-Root Account**
   - Use your login as the username.

6. **Partitioning**
   - Although you can partition manually, you can achieve optimal partitioning by choosing:
     - **Guided - Use Entire Disk and Set Up Encrypted LVM** > **Separate /home Directory**

7. **Encryption Key**
   - Set up a strong encryption password.

8. **Software Selection**
   - Ensure **SSH Server** (if not pre-selected) and **Standard System Utilities** are chosen.

---

## Install sudo
1. Login as root
2. Update package lists
	- ```bash apt update -y```
3. Update your system
	- ```apt upgrade -y```
4. Install sudo
	- ```apt install sudo -y```
5. Add your user to sudo group
	- ```adduser [user] sudo```
6. Reboot your system and login in a non-root user

---

## Setting sudo configurations

1. **Modifying sudo configuration**

   - First, create or edit the sudo configuration file:

     ```bash
     sudo visudo -f /etc/sudoers.d/sudoconfig
     ```

   - Add this line to the file:

     ```bash
     # Limit authentication attempts to 3
     Defaults        passwd_tries=3
     
     # Custom error message for wrong password
     Defaults        badpass_message="Incorrect password. Please try again."
     
     # Enable logging of both input and output
     Defaults        log_input,log_output
     Defaults        iolog_dir="/var/log/sudo"
     Defaults        logfile=/var/log/sudo/sudo.log
     
     # Enable TTY
     Defaults        requiretty
     
     # Restrict sudo paths
     Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
     ```

2. **Managing logs**

   - Create the sudo log directory if it doesn't exist:

     ```bash
     sudo mkdir -p /var/log/sudo
     ```

   - Set proper permissions:

     ```bash
     sudo chmod 700 /var/log/sudo
     ```

3. **Verify the configuration**

   - Check if the configuration is valid:

     ```bash
     sudo visudo -c
     ```

---
## Setting SSH

1. **Installation**
	- Install SSH server if not installed
	
		```bash
		apt install openssh-server
		```
2. **Configuration**
	- Configure SSH (Optional: install vim)
	
		```bash
		sudo vim /etc/ssh/sshd_config
		```

	- Edit the file:

		```bash
		# Change default port
		Port 4242
		
		# For security, disable root login
		PermitRootLogin no
		
		# Authenticate via password
		PasswordAuthentication yes
		```
3. **Apply changes**
	- Restart SSH services to apply changes

		```bash
		sudo systemctl restart ssh
		```

---
## Setting UFW
1. **Installation**
	- Install UFW
		```bash
		sudo apt install ufw
		```

2. **Configuration**
	- Basic configuration
	
		```bash
		# Reset UFW rules to default
		sudo ufw reset
		
		# Set default policies
		sudo ufw default deny incoming
		sudo ufw default allow outgoing
		```
	
	- Configure SSH Acess
	
		```bash
		# Allow SSH access only through port 4242
		sudo ufw allow 4242/tcp
		```

3. **Enabling service**
	- Enable UFW
	
		```bash
		sudo ufw enable
		```
	
	- Verify configuration
	
		```bash
		sudo ufw status verbose
		```

---

## Connect via SSH
To facilitate further steps, you can connect through SSH in your host machine.

1. **Check IP from your virtual machine**
	- Run:
		```bash
		hostname -I
		```
2. **Connect**
	- On you local machine, run:
		```bash
		ssh [login]@[server_ip] -p 4242
		```

---

## Changing password policy
1. **Installing needed library**
	- Install the required package for password quality checking:
		```bash
		sudo apt install libpam-pwquality
		```
2. **Configure password aging rules**
	- Edit this file:
		```bash
		sudo vim /etc/login.defs
		```
	- Modify/add this lines:
		```bash
		PASS_MAX_DAYS   30      # Password expires every 30 days
		PASS_MIN_DAYS   2       # Minimum days before password change
		PASS_WARN_AGE   7       # Warning 7 days before password expires
		```
3. **Configure password complexity rules**
	- Edit the PAM (Pluggable Authentication Modules) password configuration:
		```bash
		sudo vim /etc/pam.d/commom-password
		```
	- Modify/add this line:
		```bash
		password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type= minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 difok=7 reject_username enforce_for_root maxrepeat=3 no_difok=root
		```
		- `no_difok=root`: Exempts root from the `difok` requirement while keeping other rules
		- `difok=7`: Requires 7 different characters from old password (for non-root users)
		- `minlen=10`: Minimum length 10 characters
		- `ucredit=-1`: At least 1 uppercase
		- `lcredit=-1`: At least 1 lowercase
		- `dcredit=-1`: At least 1 digit
		- `maxrepeat=3`: No more than 3 consecutive identical characters
		- `reject_username`: Password can't contain username
		- `enforce_for_root`: Applies all other rules to root
	- Then, this line:
		```bash
		password    sufficient    pam_unix.so remember=7
		```
		- `remember=7`: Handles password history, preventing reuse of recent passwords.
4. **Apply the changes to existing users**
	- Run:
		```bash
		# For root user
		sudo chage -M 30 -m 2 -W 7 root
	
		# For your user (replace 'username' with your actual username)
		sudo chage -M 30 -m 2 -W 7 username
		```
	- Verify the settings:
		```bash
		# Check password policy for a user
		sudo chage -l username

		# Check root password policy
		sudo chage -l root
		```
5. **Test changes**
	- Try changing your password:
		```bash
		passwd
		```

---

## Monitoring script

1. **Creating script**
	- Create a `monitoring.sh` and copy the content of the script on this repo or make your own that checks for:
		- The architecture of your operating system and its kernel version.
		- The number of physical processors.
		- The number of virtual processors.
		- The current available RAM on your server and its utilization rate as a percentage.
		- The current available memory on your server and its utilization rate as a percentage.
		- The current utilization rate of your processors as a percentage.
		- The date and time of the last reboot.
		- Whether LVM is active or not.
		- The number of active connections.
		- The number of users using the server.
		- The IPv4 address of your server and its MAC (Media Access Control) address.
		- The number of commands executed with the sudo program.
	- Give permissions:
		```bash
		sudo chmod +X monitoring.sh
		```

2. **Add to crontab**
	- Open crontab configuration file
		```bash
		sudo crontab -u root -e
		```
	- Add the following line to the configuration (change the path):
		```bash
		*/10 * * * * /path/to/script.sh
		```
	- Save and exit

---

# Useful to know

## Creating groups and users

- Creating user 
	```bash
	sudo useradd -m [user]
	sudo passwd [user]
	```

- Creating group
	```bash
	sudo groupadd [group]
	```

- Checking groups in the machine
	```bash
	getent group
	```

- Adding user to group
	```bash
	sudo adduser [user] [group]
	```

- Deleting a user
	```bash
	sudo userdel [user]
	```

- Deleting group
	```bash
	sudo groupdel [group]
	```

- Changing hostname 
	```bash
	sudo vim /etc/hostname
	```
