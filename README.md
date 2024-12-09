# BIOENG C131/231 Final Project

This project focused on the Flavivirus family, and specifically takes a look into the variant types of Dengue viruses. Follow the below steps to create your own copy of the JBrowse2 genome browser. You can also see a working genome browser at this GitHub Page: https://ddf23.github.io or at this link (AWS hosting): http://18.117.233.155/jbrowse2/.

## 1. Browser setup (guided by Lab 8)

First, set up an instance through AWS. Once you make an account, launch an EC2 instance with an Ubuntu distribution and then connect to the instance. The IP address for your web server should be on the summary page for the instance.

### 1.1 Install Linuxbrew for AWS

Install linuxbrew: 
```
sudo su -
passwd ubuntu
```

It will prompt you to: `Enter new UNIX password:` Make sure to exit root by typing 'exit'
Install brew by using the bash script from https://brew.sh/. Enter the password made earlier when prompted. 
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Add brew to the execution path:
```
echo >> /home/ubuntu/.bashrc
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/ubuntu/.bashrc
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
```

### 1.2 Install tools. Set up Apache and JBrowse Genome Server.

Follow the instructions in https://currentprotocols.onlinelibrary.wiley.com/doi/full/10.1002/cpz1.1120) to set up JBrowse2 on your web server. This includes the section titled "Install necessary tools" and the following section "Start the Apache 2 HTTP server and use the JBrowse CLI to download JBrowse 2 to the Apache 2 web server folder." You can use the rest of the page as a guide, if necessary.

As described in the page, confirm that the installations worked by checking the link 'http://ipaddress/jbrowse2/'

## 2. Load reference genomes and genome annotations
Genome files and annotations can be taken from the NCBI Virus database. Search for the virus of interest, in this case "Dengue virus, taxid:12637". You should end up [[here](https://www.ncbi.nlm.nih.gov/labs/virus/vssi/#/virus?SeqType_s=Nucleotide&VirusLineage_ss=Dengue%20virus,%20taxid:12637)].

Extract the genome and genome annotation files of the main 4 types of Dengue virus, which have RefSeq files:
dengue virus type 1: NC_001477.1
dengue virus type 2: NC_001474.2
dengue virus type 3: NC_001475.2
dengue virus type 4: NC_002640.1

Click on the links under the "Assembly" column, whose names start with "GCF". The page you should end up in should look like this: https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000865065.1/. This example is for the type 4 dengue virus.

On that Genome Assembly ViralProj page you are led to, click the FTP link, meaning "file transfer protocol". This will take you to a folder like [[this](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/865/065/GCF_000865065.1_ViralProj15599/)] (again for type 4) where right-clicking will give you direct download links for processed files.

The files of interest are the ones that end in _genomic.fna.gz (this will be used to get an fna file that holds the virus genome) and _genomic.gff.gz (this will be used to get a gff file that holds the genome annotations).

Get the FTP link plus the names of the two files for every dengue virus type.

Once you have a direct download link, you can use wget (a linux command line tool) to download it you any machine that has internet connectivity.

Make sure you are in the temporary folder, then follow the below instructions.
```
export ROOT=insert_your_FTP_link

#loads genome assembly into jbrowse
wget $ROOT/insert_your_genomic.fna.gz
gunzip insert_your_genomic.fna.gz
mv insert_your_genomic.fna genomic_name.fa
samtools faidx genomic_name.fa
jbrowse add-assembly genomic_name.fa --out $var/www/html/jbrowse2 --load copy

#add genome annotations track into jbrowse
wget $ROOT/insert_your_genomic_annotation.gff.gz
gunzip insert_your_genomic_annotation.gff.gz
jbrowse sort-gff insert_your_genomic_annotation.gff > genomic_annotation.gff
bgzip genomic_annotation.gff
tabix genomic_annotation.gff.gz
jbrowse add-track genomic_annotation.gff.gz --out $var/www/html/jbrowse2 --load copy --assemblyNames genomic_name
```
Change the names (like genomic_annotation, genomic_name, etc.) as necessary to make unique identifiers for the different strains.

At this point, you should have four different added assemblies in JBrowse2 with their accompanying genome annotations. This can be observed using the "Linear genome view" in JBrowse2.

An extra step that could be done is to process the genome annotations. The genome annotations files that were gathered do hide more detailed information about the proteins encoded for due to them being categorized under a polyprotein gene 'parent'. By using vim to manually edit the genomic_annotation.gff files and deleting the parent id of the polyprotein gene from the relevant sub-proteins, you can follow the same instructions above to re-add genome annotation tracks that can more easily visualize the genomic annotations within the poly gene.

You can also run the below command to allow users to search by gene name within JBrowse 2.
```
jbrowse text-index --out /var/www/html/jbrowse2
```

## 3. Linear synteny view
Linear synteny view can be used to compare different assemblies together.

Refer to this [[link](https://jbrowse.org/jb2/docs/quickstart_web/)] and go to the section titled "Adding a synteny track from a PAF file". 

First install minimap2 with 'sudo apt install minimap2'
Use minimap2 to create a PAF file from two of the interested assemblies. An example to then add a synteny track for type 1 vs type2 is shown below. Be very aware of the order in which the names are listed as they can have an impact!
```
minimap2 dengue_type2.fa dengue_type1.fa > dengue_type1_vs_dengue_type2.paf
jbrowse add-track dengue_type1_vs_dengue_type2.paf --assemblyNames dengue_type1,dengue_type2 --load copy --out /var/www/html/jbrowse2
```

At this point, you can have different added synteny tracks in JBrowse2 between all the possible combinations of pairs between the viruses of interest. This can be now observed using the "Linear synteny view" or even "dotplot view" in JBrowse2. Simply select the strains you want to look at and the corresponding synteny tracks and launch the view.

## 4. MsaView
MsaView can also be used to, in more detail, compare the genomes of the viruses and see the differences between them at the nucleotide level.

First, install the MsaView plugin on JBrowse2. Go to the Tools selection at the top banner of the JBrowse2 banner, select "Plugin store", and install MsaView.

Combine your .fa files into one .fasta file and use MAFFT to perform multiple sequence alignment of the different assemblies. This results in a MAFFT fasta file with the alignment of all the input sequences.
```
cat file1.fa file2.fa file3.fa file4.fa > combined.fasta

sudo apt install mafft
mafft combined.fasta > aligned_mafft.fasta
cp aligned_mafft.fasta /var/www/html/JBrowse2
```

Input this multiple sequence alignment into FastTree to generate a Newick-formatted tree from the aligned sequence MAFFT file.

```
sudo apt install fasttree
FastTree aligned.fasta > tree.nwk
cp tree.nwk /var/www/html/JBrowse2
```

With the cp command, the two files are included into the JBrowse2 folder. This can be now observed using the "Multiple sequence alignment view" in JBrowse2. Simply, upload the MAFFT and nwk file urls (/var/www/html/JBrowse2/aligned_mafft.fasta and /var/www/html/JBrowse2/tree.nwk)in their respective spots and launch the view to see the alignments and phyologenetic tree.


# Analyze
Utilize these tools with these strains and more in the Flavivirus family to analyze and compare the genomic differences between the viruses and how they may impact pathogenicity, virality, etc.
