# BioE C131 Final Project
- Team Name: M.E.M 
- Team Members: Ekta Jaswal, Maria Isabella Solorzano, Melissa Theodorus  
- Viral Genome Family: HerpesVirus 

## Overview
This project focuses on the Herpesvirus family, particularly HSV-1, exploring the Latency-Associated Transcript (LAT) region and its regulatory interactions with flanking genes ICP0 and ICP34.5. Our database provides insights into viral latency and reactivation mechanisms, contributing to a better understanding of this virus's clinical impact.

## Features
- Focused analysis on LAT, ICP0, and ICP34.5 regions of the HSV-1 genome
- Provides a resource for studying viral replication, latency, and reactivation

## 1. AWS Set Up
### 1.1. Follow the separate AWS [setup guide](./aws_instructions.md), then return here to set up linuxbrew below.

### 1.2. Linuxbrew for WSL or AWS
Make sure you are using a Debian or Ubuntu distribution. Then go ahead and install linuxbrew, using the instructions below:

switch to root with:

`sudo su -`
Then run:

`passwd ubuntu`
It is going to prompt :

`Enter new UNIX password:`

Set your password to something you can remember for later, or write down. A common password choice is simply `ubuntu` - not very secure at all, but AWS accounts themselves can be made fairly secure. 

Exit root by typing `exit`. **Note: it is important to exit root, because you do not want to accidentally run future commands with administrator privileges when that might be undesirable. The subsequent command in this case will fail if run from root.**

Install brew using the bash script from https://brew.sh/. You will be prompted to set the password you made earlier.
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

After this is complete, add brew to your execution path:
```
echo >> /home/ubuntu/.bashrc
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/ubuntu/.bashrc
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
```
## 2. Install necessary tools
### 2.1. Node.js
Node.js is a cross-platform JavaScript runtime environment that will make is easy to run JBrowse2 command-line tools.

First, check whether Node.js is already installed by running the following. If node v20 is already installed, you can skip to the next step.

```
node -v
```

If Node.js is not installed, install it. 
### 2.2. @jbrowse/cli
Run the following commands in your shell. This uses the Node.js package manager to download the latest stable version of the jbrowse command line tool, then prints out its version. This should work for both macOS and Linux.

```
sudo npm install -g @jbrowse/cli
jbrowse --version
```

You can also try installing using just `npm install -g @jbrowse/cli` if the sudo version doesn't run. 

### 2.3. System dependencies
Install wget (if not already installed), apache2, samtools, and tabix. 

wget is a tool for retrieving files over widely-used Internet protocols like HTTP and FTP. 

apache2 allows you to run a web server on your machine.

samtools and tabix, as we have learned earlier in the course, are tools for processing and indexing genome and genome annotation files.

## 3. Apache server setup
### 3.1. Start the apache2 server
Starting up the web server will provide a localhost page to show that apache2 is installed and working correctly. When discussing computer networking, localhost is a hostname that refers to the current computer used to access the network. Note that in WSL2, the linux subsystem may have a different IP address from your Windows OS, and so you will want to use that IP address to be able to find it and load the web page. AWS, on the other hand, will have a public IP address that you need to identify in the aws_instructions.

### 3.2. Getting the host
If you are running locally on your mac, the hostname is just `localhost`. However, for WSL and AWS, you will need to do a bit of work to find the right ip address.
For local hosting, the url will be `http://localhost:8080/` or `http://XX.XXX.XXX.XX:8080/`, where Xs are replaced with the appropriate IP address from the WSL steps below.

#### WSL
```
# from within WSL, run the linux server launch command to launch the service, then print out you WSL IP address so you can access the server from your Windows browser
# if the ip command isn't recognized, install iproute and then try again
# sudo apt install iproute2
ip addr show eth0 | grep "inet\b" | awk '{print $2}' | cut -d/ -f1
```
This should give you an ip address you can use to access the web server.

#### AWS
In your instance summary page, there should be an "auto-assigned IP address." Your web server can be accessed at `http://ipaddress`. You don't need to provide a port.

### 3.3. Access the web server
Open a browser and type the appropriate url into the address bar. You should then get to a page that says "**It works!**" (for AWS there may be some additional info). If you have trouble accessing the server, you can try checking your firewall settings and disabling any VPNs or proxies to make sure traffic to localhost is allowed.

### 3.4. Verify apache2 server folder
Apache2 web servers serve files from within a root directory. This is configurable in the httpd.conf configuration file, but you shouldn't have to change it (in fact, changing the conf file is not recommended unless you know what you are doing). 

For a normal linux installation, the folder should be `/var/www` or `/var/www/html`, whereas when you install on macOS using brew it will likely be in `/opt/homebrew/var/www` (for M1) or `/usr/local/var/www` (for Intel). You can run `brew --prefix` to get the brew install location, and then from there it is in the `var/www` folder. 

Verify that one of these folders exists (it should currently be empty, except possibly for an index file, but we will now populate it with JBrowse 2). If you have e.g. a www folder with no www/html folder, and your web server is showing the "It works!" message, you can assume that the www one is the root directory. 

Take note of what the folder is, and use the command below to store it as a command-line variable. We can reference this variable in the rest of our code, to save on typing. You will need to re-run the `export` if you restart your terminal session!
```
# be sure to replace the path with your actual true path!
export APACHE_ROOT='/path/to/rootdir'
```

If you are really struggling to find the APACHE_ROOT folder, you could try searching for it.
```
sudo find / -name "www" 2>/dev/null
```

### 3.5. Download JBrowse 2
First create a temporary working directory as a staging area. You can use any folder you want, but moving forward we are assuming you created ~/tmp in your home folder.

```
mkdir ∼/tmp
```
```
cd ∼/tmp

```
Next, download and copy over JBrowse 2 into the apache2 root dir, setting the owner to the current user with `chown` and printing out the version number. This version doesn't have to match the command-line jbrowse version, but it should be a version that makes sense.

```
jbrowse create output_folder
sudo mv output_folder $APACHE_ROOT/jbrowse2
sudo chown -R $(whoami) $APACHE_ROOT/jbrowse2
```

### 3.6. Test your jbrowse install
In your browser, now type in `http://yourhost/jbrowse2/`, where yourhost is either localhost or the IP address from earlier. Now you should see the words "**It worked!**" with a green box underneath saying "JBrowse 2 is installed." with some additional details. 

## 4. Load and process test data
### 4.1. Download and process reference genome
Make sure you are in the temporary folder you created, then download the human genome in fasta format. This is the biggest file you'll be downloading, and may take 30 min or so on AWS with the lowest tier of download speeds.

```
export FASTA_ROOT=https://ftp.ensembl.org/pub/release-110/fasta/homo_sapiens
wget $FASTA_ROOT/dna/Homo_sapiens.GRCh38.dna_sm.primary_assembly.fa.gz
```

Unzip the gzipped reference genome, rename it, and index it. This will allow jbrowse to rapidly access any part of the reference just by coordinate.

```



