# DigitalOcean-Server-Setup
Guide on how to setup a droplet on DigitalOcean and add a cloud-init configuration file

## **Introduction**

In this tutorial, we will walk through the process of setting up an Arch Linux server on a remote droplet using DigitalOcean. Hopefully by the end of this guide, you'll have created a droplet with a custom Arch Linux image, automated with its initial setup using cloud-init, and securely connected to it using SSH keys.

This guide is designed for second-term CIT students, and will provide both the practical steps and underlying concepts needed to manage a remote server. We will cover key topics such as:

- **SSH keys**: A form of authentication used in the SSH protocol, functioning like a username and          password but primarily used for automated tasks.

- **SSH Protocol**: Known as Secure Shell, this protocol enables secure remote login between computers.     It offers robust authentication methods while ensuring communication security and integrity through     encryption.

- **cloud-init**: A tool for automating the configuration and setup of a new server during its first        boot.

- **DigitalOcean Droplets**: Virtual machines that can be quickly deployed and customized.

The goal of this document is not just to perform the steps but also to understand why we are doing them. By the end, we will achieved how to do the following; 

- Managing cloud infrastructure using DigitalOcean
- Using cloud-init to automate server configuration
- Implementing SSH key-based authentication for secure access

Let’s get started!

*On Windows, you might need to first create a .ssh folder in your home directory;*
		`mkdir ~/.ssh`

# **Part One: Creating a Droplet**

#### **Step 1: Creating a SSH Key on a Local Machine**

First, generate SSH keys to enable an authentication method to access the droplet.
1.  Open a Command line (preferably PowerShell), run the following command and provide a passphrase:
	   `ssh-keygen -t ed25519 -f ~/.ssh/do-key -C "your email address"`
	
	- `ssh-keygen`: This is the command used to generate SSH keys.
	- `-t ed25519`: Specifies the type of key to create, in this case, the `ed25519` algorithm                 (recommended for its security and performance).
	- `-f ~/.ssh/do-key`: This specifies the location and name of the key file. Here, the key will be           saved as `do-key` in the `.ssh` directory under your home folder.
	- `-C "your email address"`: Adds a comment to the key to help identify it, typically your email            address.
	
2.  Next, on the terminal, apply this command;
	   `Get-Content C:\Users\your-user-name\.ssh\do-key.pub | Set-Clipboard`
	   
	- `Get-Content C:\Users\your-user-name\.ssh\do-key.pub`: This part reads the contents of the public         key file (`do-key.pub`) located in the `.ssh` directory of your user profile.
	    
	- `|`: This is a pipeline operator that takes the output of the command on the left (in this case,         the contents of the public key) and passes it as input to the command on the right.
	    
	- `Set-Clipboard`: This command takes the input it receives (the public key content) and copies it to      the clipboard, allowing you to easily paste it elsewhere.

#### **Step 2: Add SSH Key to Digital Ocean Account**

After generating your SSH key, you need to add it to your DigitalOcean account.

1. Log in to your DigitalOcean account.
2. Navigate to **Settings --> Security**.
3. Under "SSH Keys," click **Add SSH Key**.
4. Paste the contents of your **public key** `C:\Users\<your-username>\.ssh\id_rsa.pub` into the text box, add a name to the key and click **Add SSH Key**.
   
#### **Step 3: Add a Custom Arch Linux Image Using the DigitalOcean**

1. Download the official Arch Linux ISO from the Arch Linux website. 
		https://gitlab.archlinux.org/archlinux/arch-boxes/-/packages/
2. In DigitalOcean, go to **Images** > **Custom Images**.
3. Click **Upload Custom Image**.
    - Choose your Arch Linux ISO.
    - Set the region for your droplet.
    - Select **Arch Linux** as the image type.
4. Wait for the image to upload.
   
#### **Step 4: Create a Droplet Running Arch Linux**
1. Once logged in, there will be a dropdown button **Create** at the top of the page. 
2. Click the button and the menu will dropdown displaying a **Droplets** option. 
3. Selecting that option will lead you to a new page titles Create Droplets
   
	 1. **Chose Region**: Pick San Francisco (SFO3) as the default data center due to it being the                nearest location to us.
	 2. **Choose an image**: A navigation bar depiction three options; 
			 -  OS
			 -  Marketplace (246)
			 - Custom images (this is where the Arch Linux image we uploaded will reside)
	3. Select Custom image and pick our uploaded Arch Linux image. 
	4. **Choose Size**: For our uses, the default Basic Plan will suit us fine as the default droplet            type.
	5. For CPU options we will select **Premium Intel** Disk: NVMe SSD for $8 a month (or $0.012/hour),         this includes 35 GB for your storage.
	6. **Choose Authentication Method**: This is where the SSH key comes into play, select the key you           added
	7. Give the droplet a hostname like arch or bcit to make it shorter when it appears on the prompt.
	8. All other settings can be left on default

Now for the cloud-init file, leave the droplet creation page as it is for now, we will be adding an initialization script.

# **Part Two: Adding the Cloud-Init Configuration File**

**Cloud-init** is a widely used tool for initializing cloud instances automatically during the boot process. It will allow us to configure and customize cloud instances at the time of deployment. 

### **How Cloud-Init Works**

1. **Initialization at Boot**: When a cloud instance is launched, cloud-init runs automatically during       the initial boot process. It retrieves metadata from the cloud provider, such as user data and          instance-specific information.
    
2. **User Data**: Cloud-init can be configured using "user data," which is typically provided as a           script or configuration file when launching the instance. This user data can include files such as      cloud-config YAML files.
    
3. **Stages of Execution**: Cloud-init operates in several stages:
    
    - **Init**: The initial stage where cloud-init collects metadata from the cloud provider.
    - **Config**: The configuration stage, where it processes the user data and applies any specified         settings.
    - **Final**: The final stage executes any commands specified in the user data.

4. **Handling Metadata**: Cloud-init retrieves instance metadata from the cloud provider (e.g., AWS,         Azure, DigitalOcean), which contains information about the instance, such as its network                configuration, region, and security settings.

#### **Step 5: Setting up a Cloud-Init Configuration File**

- Now lets create the config file to automate the setup of our Arch Linux droplet
- We'll start by creating a file that can be named as cloud-config.yaml
	- YAML is a serialization language, a lot like JSON and a superset of it, meaning that it is often        used for writing configuration files
- You can use notepad to write the document but doing this part in a text-editor is much better and you   are student of information technology. 

#### **Step 6: Writing the Cloud-Init File**

- With the file now created and ready to be edited, add the following content.

`#cloud-config`  

`users:`
	`- name: newuser` 
	   `ssh-authorized-keys: "ssh-rsa your-public-key"`
	   `sudo: ['ALL=(ALL) NOPASSWD:ALL']`
       `shell: /bin/bash`
`write_files:`
	`- path: /home/newuser/welcome.txt`
	  `contents: |`
		`Welcome to your new Arch Linux server, you have successfully created the droplet and initialized a cloud-init configuration file. Well done!` 
`runcmd:` 
	`- echo "Welcome file successfully created"`

###### Explanation
- `#cloud-config` 
	- This line is the file header, indicating that its a cloud-init configuration file for cloud-init to process it correctly.
- `users:` This section is the first directive key which defines user accounts, the authorized key of     the user, sudo privileges, and the shell to be used as a single item value under the directive. This    will tell the cloud-init what to add to or use in the account. For example, here a new user is being    created called newuser, they don't require password access to sudo, and the shell they will using.
- `write_files:` This second directive informs what the file's path be, and its contents. This file       includes `|` character is used for a multiline string
- `runcmd`: This last directive ends the file by executing a command once the config file is               initialized and processed. In this case it echoes a message stating that file has been created          successfully.

#### **Step 7: Adding the Configuration File to the Droplet**

Now that the config file has been made, we must add it to the droplet.

- Under the **Choose Authentication Method**, click on advanced options and  choosing the **Add           Initialization scripts**
- A text box will display soon after, here we paste the contents of the cloud-init config file
- Finally, we can click on **Create Droplet** on the bottom right of the screen.

#### **Step 8: Connecting to the Droplet**

This is the last part of process and perhaps the most important. Connecting to our Linux server. 

- Open the Window's terminal and run the following command;
		`ssh -i .ssh/do-key arch@your-droplets-ip-address`

  	-  `-i`: This option is used to specify the identify file, which here is the private SSH key (`do-          key`) for authentication. 
  	- `arch@your-droplets-ip-address`: This specifies the username (`arch`) and the IP address of the         remote server (`your-droplets-ip-address`). 
  	- Replace `your-droplets-ip-address` with the actual IP address of your DigitalOcean droplet.

#### **Troubleshooting** 
- **SSH connection error**: If you receive a "Permission denied" error when connecting via SSH, make       sure that: 
  	- The SSH key was correctly added to DigitalOcean. 
  	- You are using the correct private key (`-i` flag). 
  	- The correct username (`arch`) and IP address are used.


##### **Citations and External Resources** 

SSH Communications Security. (n.d.). _SSH keys: What they are and how they work_. SSH Academy. Retrieved September 19, 2024, from [https://www.ssh.com/academy/ssh-keys](https://www.ssh.com/academy/ssh-keys)
    
DigitalOcean. (2020, March 11). _An introduction to cloud-config scripting_. DigitalOcean. Retrieved September 27, 2024, from [https://www.digitalocean.com/community/tutorials/an-introduction-to-cloud-config-scripting](https://www.digitalocean.com/community/tutorials/an-introduction-to-cloud-config-scripting)
    
Cloud-Init Documentation. (n.d.). _Cloud-init documentation_. Read the Docs. Retrieved September 20, 2024, from [https://cloudinit.readthedocs.io/en/latest/](https://cloudinit.readthedocs.io/en/latest/)
    
Arch Linux. (n.d.). _Cloud-init_. ArchWiki. Retrieved September 20, 2024, from [https://wiki.archlinux.org/title/Cloud-init](https://wiki.archlinux.org/title/Cloud-init)
    
Red Hat. (n.d.). _What is YAML?_. Red Hat. Retrieved September 20, 2024, from https://www.redhat.com/en/topics/automation/what-is-yaml
