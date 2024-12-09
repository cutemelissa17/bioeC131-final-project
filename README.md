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

## 4. Load and process data
### 4.1. Download and process reference genome
Make sure you are in the temporary folder you created, then download the HSV-1 genome in fasta format. This is the biggest file you'll be downloading, and may take 30 min or so on AWS with the lowest tier of download speeds.

```
wget -O herpesvirus_dataset.zip "https://api.ncbi.nlm.nih.gov/datasets/v2/genome/accession/GCF_000859985.2/download?include_annotation_type=GENOME_FASTA&include_annotation_type=GENOME_GFF&include_annotation_type=RNA_FASTA&include_annotation_type=CDS_FASTA&include_annotation_type=PROT_FASTA&include_annotation_type=SEQUENCE_REPORT&hydrated=FULLY_HYDRATED"
```
### 4.2 Unzip the gzipped reference genome, rename it, and index it. This will allow jbrowse to rapidly access any part of the reference just by coordinate.

```
unzip herpesvirus_dataset.zip -d herpesvirus_data
cd herpesvirus_data
```
### 4.3 Rename the genome file and create an index:
```
mv ./ncbi_dataset/data/GCF_000859985.2/GCF_000859985.2_ViralProj15217_genomic.fna herpesvirus1.fa
samtools faidx herpesvirus1.fa
```
### 4.4 Add Genome and Tracks to JBrowse2
```
jbrowse add-assembly herpesvirus1.fa --out $APACHE_ROOT/jbrowse2 --load copy
```
### 4.5 Sort, compress, and index the GFF file:
```
jbrowse sort-gff ./ncbi_dataset/data/GCF_000859985.2/genomic.gff > herpesvirus1_sorted.gff
bgzip herpesvirus1_sorted.gff
tabix herpesvirus1_sorted.gff.gz
```
### 4.6 Add the annotation track to JBrowse2:
```
jbrowse add-track herpesvirus1_sorted.gff.gz --name "Herpesvirus Annotations" --out $APACHE_ROOT/jbrowse2 --load copy
```

## 5. Explore LAT and Flanking Regions
### 5.1 Create a BED file for LAT regions:
```
echo -e "NC_001806.2\t1\t7569\tLAT_region_1" > lat_regions.bed
echo -e "NC_001806.2\t118777\t127151\tLAT_region_2" >> lat_regions.bed
bgzip lat_regions.bed
tabix lat_regions.bed.gz
```
### 5.2 Add LAT regions as a track:
```
jbrowse add-track lat_regions.bed.gz --name "LAT Regions" --out $APACHE_ROOT/jbrowse2 --load copy
```
## 5.3 Add Flanking Genes
Create a BED file for RL1 and RL2 flanking genes:
```
echo -e "NC_001806.2\t120000\t121000\tICP34.5" > rl_genes.bed
echo -e "NC_001806.2\t118000\t119000\tICP0" >> rl_genes.bed
```
### 5.4 Sort the BED File:
```
sort -k1,1 -k2,2n rl_genes.bed > rl_genes_sorted.bed
```
Compress and Index the BED File
```
bgzip -f rl_genes_sorted.bed
tabix -f rl_genes_sorted.bed.gz
```
Add track to jbrowse
```
jbrowse add-track rl_genes_sorted.bed.gz --name "Flanking Genes (ICP34.5 & ICP0)" --out $APACHE_ROOT/jbrowse2 --load copy
```
### 6 LAT-Gene Proximity Final Track
## 6.1. Generate the LAT Regions File:

```
echo -e "NC_001806.2\t1\t7569\tLAT_region_1\tregulatory_role=latency_promotion" > lat_regions_with_metadata.bed
bgzip -f lat_regions_with_metadata.bed
tabix -f lat_regions_with_metadata.bed.gz
```

## 6.2 Proximity Analysis Using Bedtools:
```
bedtools closest -a lat_regions_with_metadata.bed.gz -b rl_genes_sorted.bed.gz > lat_gene_proximity.bed

bgzip -f lat_gene_proximity.bed
tabix -f lat_gene_proximity.bed.gz

jbrowse add-track lat_gene_proximity.bed.gz --name "LAT-Gene Proximity Final" --out $APACHE_ROOT/jbrowse2 --load copy
```
### 7 RNA Structure Analysis with ViennaRNA

## 7.1 Install ViennaRNA and Bedtools
```
sudo apt-get update
sudo apt-get install vienna-rna bedtools
```
## 7.2 Extract Sequences from BED File
```
bedtools getfasta -fi /var/www/html/jbrowse2/herpesvirus1.fa -bed /var/www/html/jbrowse2/lat_regions_updated.bed -fo lat_regions_sequences.fasta
```

## 7.3 #Preparing Data for RNA Structure Prediction
```
RNAfold < lat_regions_sequences.fasta > lat_regions_structure.txt
Visualizing the RNA Structure Using the RNAfold Web Server
Result:  http://rna.tbi.univie.ac.at//cgi-bin/RNAWebSuite/RNAfold.cgi?PAGE=3&ID=2Dum_KBGoe
```
Open `http://yourhost/jbrowse2/` again in your web browser. There should now be several options in the main menu. Follow the guide in the "Launch the JBrowse 2 application and search for a gene in the linear genome view" section of https://currentprotocols.onlinelibrary.wiley.com/doi/10.1002/cpz1.1120 to navigate to the gene search and try browsing a few genes.

# Submission
Commit your read file with the prompts answered, then push to your remote repository and provide the link to your repo in the bCourses submission form.
