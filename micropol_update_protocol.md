% Author : Minju Lee

Micropollutant data management platform for wastewater treatment processes using Python 2.7 and SQLite3
========

## A. Preface
This suite of modules written in Python 2.7 was developed to provide a database platform where one can store, manage, and utilize the datasets of micropollutants in the context of their occurrence and fate during water/wastewater treatment processes.

The basic package comprises

* db_creation.py
* db_micropols.py
* db_campaigns.py
* db_datafetch.py

Two excel files (.xlsx) below serve as inputs for the Python modules, which contain micropollutants and treatment processes-relevant information.

* micropol_info.xlsx
* micropol_campaigns.xlsx

One can prepare raw data in these excel file and update the information using an appropriate Python module. Filenames and their contents can be modified freely, however, it should be noted that any modification of the content format such as the headers of columns and the names of excel worksheets can incur conflicts between the input files and the Python programs unless the codes are appropriately adapted.

In addition, extension modules below are available, which are to be utilized to update physico-chemical and kinetic (e.g., ozone and hydroxyl radical rate constants) information of micropollutants and based on such information to perform a prediction method for the removal of micropollutants during ozonation using indicator compounds.

* db_microspecies.py
* micropol_microspecies.xlsx (as input for db_microspecies.py)
* o3_apparent_rate.py
* o3_pathway.py
* o3_indicat_model.py
* o3_indicat_plot.py

It should be noted that the functionalities of some modules above will not be available for a user because it requires additional installation of in-house and licensed softwares and are not planned yet to be released to the public. The input excel files, which contain data for numeorus micropollutants (>600) pre-generated using such modules, will be provided instead.   

Although ozonation is currently only available as an appropriate prediction method has been developed by the author, it is of course possible to extend modules for other treatment processes in the future when appropriate methods are available.  

Detailed descriptions on each module follow below. Note that all the options and relevant information for the Python modules described above can be found with -h option, e.g.,
~~~ bash
db_micropols.py -h
~~~

## B. Basic package
### B.1. Database creation (db_creation.py)

Create a database (.db) named by `dbname`.

~~~ bash
python db_creation.py -d dbname
~~~
The command above creates a database in structures pre-defined by the author and can be examined using sqlite3 as follows.
~~~ bash
sqlite3 dbname
~~~
Refer to the link https://www.sqlite.org/ for instructions as to how to handle SQLite database.
Note that, as mentioned above, any structural modification of the database will incur conflicts.

The database has 13 tables in total as follows and all the information across the 13 tables are appropriately connected via common keys between related tables. The visual connectivity between the tables of the database is shown in Figure 1 below. A brief introduction about the structure and the contents of the tables follows below.
Looking at the following input files together with the following descriptions would be helpful to understand.   
  `micropol_campaigns.xlsx` for campaign-, treatment process-, and treatment data-relevant tables,   `micropols_info.xlsx` for the `micropols` table,  
  `micropols_microspecies.xlsx` for the `microspecies` table

**● Campaign-relevant tables**
+ **campaigns**: authors, year, reference of publications (academic journal or technical report), and URL
+ **wwtp**: city, country, wastewater type (e.g., municipal, industrial), discharge (m<sup>3</sup>/d), coordinates (longitude, latitude)

**● Treatment process-relevant tables**

+ **biological**: treatment information about biological treatment
                  type (e.g., activated sludge, MBR), scale (e.g., lab, pilot, full), date/period(YYYY-MM-DD/YYYY-MM-DD-YYYY-MM-DD), solid retention time (day), hydraulic retention time (hours), flow (m<sup>3</sup>/d), miscellaneous info.
+ **ozonation**: treatment information about ozonation
                 injection type (e.g., gas, liquid), scale (lab, pilot, full), date/period (YYYY-MM-DD/YYYY-MM-DD-YYYY-MM-DD), dose (mg/L), dose (gO<sub>3</sub>/gDOC), flow (m<sup>3</sup>/d), sandFilter_included (True/False)
                 miscellaneous info.
                 \*`sandFilter_included` is to check whether treatment data such as removals or concentrations corresponding to the ozonation information updated in this table includes the results of both ozonation and the post-sand filtration or not. It is because while some studies report only the treatment data including both treatments, some studies report separately, in which case the treatment data for post-sand filtration is to be updated in the `sandFiltration` table below.

+ **sandFiltration**: treatment information about post sand filtration following ozonation
                      scale (lab, pilot, full), date/period (YYYY-MM-DD/YYYY-MM-DD-YYYY-MM-DD), flow (m<sup>3</sup>/d), miscellaneous info.

+ **ozonation**: treatment information about ozonation
                injection type (e.g., gas, liquid), scale (lab, pilot, full), date/period (YYYY-MM-DD/YYYY-MM-DD-YYYY-MM-DD), dose (mg/L), dose (gO<sub>3</sub>/gDOC), flow (m<sup>3</sup>/d), sandFilter_followed(True/False)
                miscellaneous info.
+ **granularAC**: treatment information about granular activated carbon
                  GAC product info (e.g., CARBOPAL®, Dounau Carbon, Germany), scale (lab, pilot, full), date/period (YYYY-MM-DD/YYYY-MM-DD-YYYY-MM-DD), bed volume (m<sup>3</sup>/m<sup>3</sup>), flow (m<sup>3</sup>/d), empty bed contact time (min), filter velocity (min/hr),miscellaneous info.
+ **poweredAC**: treatment information about powdered activated carbon
                 PAC product info (e.g., Norit SAE Super), scale (e.g., lab, pilot, full), date/period (YYYY-MM-DD/YYYY-MM-DD-YYYY-MM-DD), dose (mg/L), dose (gPAC/gDOC), flow (m<sup>3</sup>/d), hydraulic retention time (hours), PAC retention time (day), circulation to biological (True/False), miscellaneous info.

+ **waterquality**: water quality information for each treatment process
           water type(influent/effluent of the treatment process), pH, temperature(celcius), DOC (mgC/L), alkalinity (mg/LasHCO<sub>3</sub>), total nitrogen (mgN/L), ammonia (mgN/L), nitrate (mgN/L), nitrite (mgN/L), bromide (\mug/L), UV<sub>254nm</sub>

 **● Treatment data-relevant tables**
 + **concentrations**: treatment data about concentrations of micropollutants
                       influent (ug/L), influent error as standard deviation (ug/L), effluent (ug/L), effluent error as standard deviation (ug/L).
                       \* Note that influent/effluent concentrations can be given not only as a definite value but also as a range such as <10, >3000
 + **removals**: treatment data about removals of micropollutants
                 removal (%), removal error as standard deviation (%)
                 \* Note that removals can be given not only as a definite value but also as a range such as <10, >95

**● Micropollutants-relevant tables**
+ **micropols**: information about micropollutants
                 compound name, abbreviation, SMILES string, CAS number, substance group (e.g., pharmaceutical, pesticide, etc), substance subgroup (e.g., antibiotic, fungicide), classification (e.g., transformation products), parent compound in case the micropollutant is a transformation product, kOH, reference information such as kOH_ref and kOH_URL for the kOH value
                 \* kOH, kOH_ref, kOH_URL are the kinetic information for the reaction of a micropollutant with hydroxyl radicals which are additionally given.
+ **microspecies**: information about the microspecies of micropollutants
                    experimental and predicted pka, tautomeric fraction, ozone rate constants.

+ **ko3_site**: information about the predicted site-specific ozone rate constants (ko3_site_pred) for the microspecies of a micropollutant.  
 site (e.g., phenol, amine, etc.), the list of atom numbers for the site (e.g., 33,34,35,36,37,38 for phenol, 5 for amine), ko3_site_pred.
 \*All the information in this table are produced by running a in-house prediction package for ozonation.


### B.2. Mircopollutant updates

| Python module | Input file      | Database tables |
|:-------------:|:---------------:|:---------------:|
| db_micropol.py| micropols.xlsx  | micropols       |


Update a micropollutant table (`micropols`) with the information found in an input file. By default, it reads in the information in `micropols` worksheet in the `micropols.xlsx` file.

~~~ bash
python db_micropols.py -d dbname -i micropols.xlsx
~~~

`micropols.xlsx` currently contains more than 600 micropollutants relevant to the aquatic environment and their basic information such as name, SMILES, CAS number, classifications, etc.
Note that the `micropols` table in a database has fields for kinetic information for the reaction of a micropollutant with hydroxyl radicals such as `kOH`, `kOH_ref`, `kOH_URL`. These fields are to be dealt with using `o3_microspecies.py` module (see below).

### B.3. Campaign data updates (db_campaigns.py)

| Python module  | Input file               | Database tables                                               |
|:--------------:|:------------------------:|:-------------------------------------------------------------:|
| db_campaigns.py| micropols_campaigns.xlsx | campaigns, wwtp, biological, ozonation, granularAC, powderedAC, sandFiltration|

Update campaign-relevant tables such as `campaigns`, `wwtp`,`biological`, `ozonation`, `granularAC`, `powderedAC`, and `sandFiltration` with the information contained in the worksheet of the same name in `micropol_campaigns.xlsx` file. Note that, in the treatment worksheets such as `biological`, `ozonation`, `granularAC`, `powderedAC`, and `sandFiltration`, `campaign_key` and `wwtp_key` serve as a key in the database, connecting the treatment information to the corresponding campaign and wwtp, respectively. For example, the campaign key 'hollender' in the `ozonation` worksheet connects to the campaign info in the the 'campaign' worksheet with the same campaign key. In the same manner, wwtp_key is used for wwtp table.

~~~ bash
python db_campaigns.py -d dbname -i micropol_campaigns.xlsx -t treatment
~~~

Furthermore, `data_key` in a treatment worksheet connects each treatment process to the corresponding treatment data table (e.g., `removals` and `concentrations`), which can be updated as follows.

~~~ bash
python db_campaigns.py -d dbname -i micropol_campaigns.xlsx -t treatment
~~~

Each table except `removals` and `concentrations` can be entirely exported in a csv file with tab ('\t') as delimiter as follows.

~~~ bash
python db_campaigns.py -d dbname -e -t treatment -o filename
~~~

### B.4. Data fetch from database (db_datafetch.py)

Fetch a desired set of data from the database across all the tables mentioned above, i.e., campaign-related tables and micropol table. As shown below, there are many options required in order to fetch the customized dataset. Use -h option to find the detailed instructions for each option.

~~~ bash
python db_datafetch.py -d dbname -t treatment -c campaign_key -i data_key -q waterquality -w wwtp -r result -o filename
~~~

## C. Extension package
### C.1. Update information about microspecies of a micropollutant (db_microspecies.py)

In interpreting treatment data for micropollutants during water/wastewater treatments, treatment-relevant physico-chemical or kinetic information of their microspecies may be useful where microspecies are referred to as a group of chemical species for a micropollutant in differing ionization states depending on its ionizing functional groups. In the current version, the `microspecies` table was made to contain the following properties which are mainly intended to be utilized for the ozonation process.  

+ **SMILES**: SMILES string representing the two-dimensional chemical structure of a microspecies. Microspecies for a micropollutant can be automatically generated using Chemaxon pKa predictor, which needs to be separately installed by a user.
+ **net charge**: net charge of a microspecies
+ **pka_pred**: acid-dissociation constant predicted using Chemaxon pKa predictor
+ **tautomeric_frac_pred**: tautomeric fraction of a microspecies predicted using Chemaxon pKa predictor, which needs to be installed by a user
+ **pka_exp**: experimental acid-dissociation constant of a micropollutant
+ **pka_exp_ref**: reference for the experimental acid-dissociation constant
+ **pka_exp_URL**: URL for the experimental acid-dissociation constant
+ **tautomeric_frac_exp**: experimental tautomeric fraction of a microspecies. Currently, it is assumed to be referred by the same reference as the one for pKa

+ **ko3_pred**: second-order rate constant (/M/s) for the reaction of a microspecies with ozone predicted using a prediction method developed by `Lee M, Blum LC, Schmid E, Fenner K, von Gunten U. A computer-based prediction platform for the reaction of ozone with organic compounds in aqueous solution: kinetics and mechanisms. Environ Sci Process Impacts. 2017`, URL: http://xlink.rsc.org/?DOI=C6EM00584E

+ **ko3_exp**: experimental ozone rate constant(/M/s)
+ **ko3_exp_ref**: reference for the experimental ko3
+ **ko3_exp_URL**: URL for the experimental ko3

Moreover, **kOH**, which is a second-order rate constant for the reaction of a micropollutant with hydroxyl radicals, is to be updated in the `micropols` table rather than the `microspecies` table. This is based on the assumption that **kOH** is the same for all the microspecies of a micropollutant.

There are four main options for this modules, *g*, *e*, *u*, and *k*. The ideal utilization of this module would be initialized with *g* option with an input file which provides a list of micropollutants to update the table as follows.

~~~ bash
python db_microspecies.py -d dbname -g -i micropols.xlsx -q querytype
~~~

The command above generates all the microspecies of a micropollutant, potentially relevant to the pH range between -3 and 17 using the Chemaxon pKa predictor and store their net charge, SMILES, pka_pred, and tautomeric_frac_pred, all of which obtained on the fly by the pKa predictor, into the `microspecies` table. However, this module will not be available to a user because as mentioned above the third-party softwares required for this module will not be provided. Instead, the `micropols_microspecies.xlsx` file given to a user contains such predicted information as well as some experimental values if available for about 640 micropollutants in `micropols.xlsx`. Therefore, a user can update the other fields related to the experimental values and associated references.  
If a user wants to insert entries for microspecies of new micropollutants, one can simply add as many as rows to the `micropols_microspecies.xlsx` file with the corresponding information and update with the *u* and *t* options with the 'microspecies' argument for the latter as below.

~~~ bash
python db_microspecies.py -d dbname -u -t microspecies -q querytype -i micropols.xlsx
~~~

While *e* and *u* options can be operated without third-party softwares, *k* option, that is to perform ko3 prediction, will not be available for the same reason as the *g* option as described above. Nonetheless, the predicted ko3 values for the microspecies of about 640 micropollutants in `micropols.xlsx` are provided.

### C.2. Derive an apparent ozone rate constant at specific pH (o3_apparent_rate.py)

### C.3. Predict ozone reaction pathways and the ensuing transformation products (o3_pathway.py)

### C.4. Implementation of an indicator-concept method for predicting abatement efficiencies of micropollutants during ozonation of water/wastewaters (o3_indicat_model.py)

### C.5. Plotting results of an indicator-concept method (o3_indicat_plot.py)


+ **/** which is the root partition (contains everything under it)
+ **swap** which prevents the server from crashing when the memory limit is
    exceeded. (it's also used to shift some content from memory to swap (disk)
    when it's not used and other content would need the place).

More partitions could be useful :

+ **/clone_sys** which we use to install a dual boot with Clonezilla
    (used to make cold image of the server). This partition needs 1GB.

NB. It's a good idea to put the partition that might one day need extension as
the last partition.

### What to remember once the server is installed ?

* Security updates have to be applied on a regular basis. Chapter B) gives the
way to do it.


INSTALL
=======

2017-10-02 ● OS Install
------------------------------------------------------------------------

Download Ubuntu 16.04 server 64bits from
[here](http://mirror.switch.ch/ftp/ubuntu-cdimage/16.04/ubuntu-16.04-server-amd64.iso)

* Boot on ubuntu-16.04-server-amd64.iso

\  

* Language : `English`
* Keymap : `F3 -> fr_CH`
* Country : `Europe/Switzerland`
* Locale : `en_US.UTF-8`

\  

* Network configured manually :
    * IP : `128.178.240.82`
    * Netmask : `255.255.255.0`
    * Default Gateway : `128.178.240.1`
    * Name server addresses (DNS)  : `128.178.15.8 128.178.15.7`
    * Hostname : `ltqevm1`
    * Domain name : `epfl.ch`

\  

* New user :
    * Full name : `Minju Lee`
    * username : `mlee`
    * pwd : `_choosen pwd_`
    * Encrypt home : `No`

\  

* Timezone : `Europe/Zurich`

\  

* Partionning configured manually :
    * If needed, select and hit \<enter\> on the disk to create an empty partition table.
    * select and hit \<enter\> on the "FREE SPACE" to create new partions :

        |    | type    | partition | size        | filesystem | mount point  |
        | -- | ------- | --------- | ----------- | ---------- | ------------ |
        | -> | primary | `sda1`    | `_1_ GB`    | `ext4`     | `/clone_sys` |
        | -> | primary | `sda2`    |  `_115_ GB` | `ext4`     | `/`          |
        | -> | logic   | `sda5`    |  `_4_ GB`   | `swap`     |              |

    Note : See question "How to partition my server" in Preamble section.

\  

* Proxy : `none`

\  

* `No automatic updates`

\  

* Software to install :
    * `standard system utilities`
    * `OpenSSH server`

\  

* Install GRUB in MBR : `YES`


2017-10-02 ● OS Updates
------------------------------------------------------------------------

~~~ bash
sudo apt update && sudo apt dist-upgrade; /usr/lib/update-notifier/update-motd-reboot-required
sudo reboot && exit
~~~

The 1st command is a shortcut to fetch the list of the updates available; apply
them (after listing them and asking the admin his agreement); and finally
display a message if the reboot is required (or no message).

The second reboots the server and exit the terminal. It's useful to exit the
terminal before so that bash history is saved.


2017-10-02 ● Server Basics
------------------------------------------------------------------------

Common packages. (pick those you need)

### ENACdrives

Mount/umount EPFL's NAS on the server[http://enacit.epfl.ch/enacdrives/](http://enacit.epfl.ch/enacdrives/)

~~~ bash
sudo vi /etc/apt/sources.list.d/enacrepo.list
~~~

~~~ snip
# http://enacit1.epfl.ch/linux/sys/enacrepo.shtml
deb http://enacrepo.epfl.ch/public xenial main    # pour Ubuntu 16.04 LTS
~~~

~~~ bash
wget -q http://enacrepo.epfl.ch/enacrepo.asc -O- | sudo apt-key add -
sudo apt update && sudo apt install enacdrives
vi /etc/enacdrives.conf
~~~

~~~ snip
[global]
Linux_CIFS_method = mount.cifs
~~~

### Vi Improved

Edit files from command line with vi (which alias to vim)
[https://en.wikipedia.org/wiki/Vim_%28text_editor%29](https://en.wikipedia.org/wiki/Vim_%28text_editor%29)

~~~ bash
sudo apt install vim
~~~

### tree

Nice output of a whole tree of files
[http://www.computerhope.com/unix/tree.htm](http://www.computerhope.com/unix/tree.htm)

~~~ bash
sudo apt install tree
~~~

### screen

Use multiple shell windows and keep them active even after logout (alternative to tmux)
[https://www.rackaid.com/blog/linux-screen-tutorial-and-how-to/](https://www.rackaid.com/blog/linux-screen-tutorial-and-how-to/)

~~~ bash
sudo apt install screen
~~~

Notes :
screen manager
-----------------------------------------------------------------------------------------------------------------

config file : ~/.screenrc

screen -S nom-session           # create a session
screen -d [nom-session]         # detach from session
screen -ls                      # list sessions
screen -r [nom-session]         # re-atach to session (detached one)
screen -r -x [nom-session]      # idem but not detached one (multi display mode)

C-a ?           # help
C-a d           # Detach
C-a C-c         # create a new tab and jump to it
C-a C-k         # destroy current tab
C-a A           # set tab's name
C-a 1           # jump to tab number 1 (or x)
C-a C-a         # jump to previous tab
C-a [           # start moving up & down (scroll)
    ]           # ends moving
C-a C-x         # lock terminal
C-a F           # Resize the window to current region size
C-a \           # kill all windows and terminate screen


### multitail

Follow several logs in one console
[http://www.tecmint.com/view-multiple-files-in-linux/](http://www.tecmint.com/view-multiple-files-in-linux/)

~~~ bash
sudo apt install multitail
~~~

### Glances

System monitoring tool
[https://nicolargo.github.io/glances/](https://nicolargo.github.io/glances/)

~~~ bash
sudo pip3 install Glances
~~~

### iftop

Bandwidth usage monitoring
[http://www.ex-parrot.com/pdw/iftop/](http://www.ex-parrot.com/pdw/iftop/)

~~~ bash
sudo apt install iftop
~~~

### dstat

Versatile resource statistics tool
[http://dag.wiee.rs/home-made/dstat/](http://dag.wiee.rs/home-made/dstat/)

~~~ bash
sudo apt install dstat
~~~

### essential packages for compilation

~~~ bash
sudo apt install build-essential
~~~

### Meld

Browse differences between 2 or 3 files or folders (and be able to merge them)
[http://meldmerge.org/](http://meldmerge.org/)

~~~ bash
sudo apt install meld
~~~

### git

distributed version control system
[https://git-scm.com/](https://git-scm.com/)

~~~ bash
sudo apt install git
~~~

### Python 2

[https://docs.python.org/2/](https://docs.python.org/2/)

~~~ bash
sudo apt install python-dev python-pip
~~~

### Python 3

[https://docs.python.org/3/](https://docs.python.org/3/)

~~~ bash
sudo apt install python3-dev python3-pip
~~~

### Python Virtualenv

[http://docs.python-guide.org/en/latest/dev/virtualenvs/](http://docs.python-guide.org/en/latest/dev/virtualenvs/)

~~~ bash
sudo apt install virtualenv
~~~


2017-10-02 ● Additional admin users on the server
------------------------------------------------------------------------

For ETH-SIS - emanuel.schmid@id.ethz.ch

~~~ bash
sudo groupadd eth-sis
sudo useradd -m -c "ETH Scientific IT Services" -g eth-sis -G adm,cdrom,sudo,dip,plugdev,lxd,lpadmin,sambashare -s /bin/bash eth-sis
sudo passwd eth-sis
~~~

2017-10-02 ● Additional non-admin users on the server
------------------------------------------------------------------------

You can have multiple non-admin users ... Not used

~~~ bash
sudo groupadd _username_
sudo useradd -m -c "_Full User Name_" -g _username_ -s /bin/bash _username_
sudo passwd _username_
~~~


2017-10-02 ● VMwareTools
------------------------------------------------------------------------

This only applies to VMware virtual machines

~~~ bash
sudo apt install open-vm-tools
~~~


2017-10-02 ● Mail config
------------------------------------------------------------------------

This is useful for the case the server wants to notify the admins of a problem,
like an error while running a cron or whatever else that would use the command
`mail`. You, as admin, might also want to use that command `mail` in your
scripts.


Note : Sending emails at EPFL requires authentication (using port 465 SSL/TLS).
However if you don't want to use an account and store username + password on the
server, you can use port 25 (no authentication) and refer in the `_FROM_` field
to a service account ([http://services.epfl.ch/](http://services.epfl.ch/))
redirected to `_noreply@epfl.ch_`.

To simplify the procedure, one can use the default service account named
`_noreply@epfl.ch_` or a customized one like `_noreply+anything-here@epfl.ch_`
which is equivalent to the first one.

Here is how to set it up :

~~~ bash
sudo apt install mailutils ssmtp
sudo vi /etc/ssmtp/ssmtp.conf
~~~

~~~ snip
root=Minju.Lee@epfl.ch
mailhub=mail.epfl.ch
hostname=ltqevm1.epfl.ch
~~~

~~~ bash
sudo vi /etc/ssmtp/revaliases
~~~

~~~ snip
root:noreply+ltqevm1@epfl.ch:mail.epfl.ch
~~~

~~~ bash
echo test | sudo mail -s test1 Minju.Lee@epfl.ch
~~~


2017-10-02 ● NTP
------------------------------------------------------------------------

NTP adjusts the server's time to the time reference servers.

~~~ bash
sudo apt install ntp
sudo vi /etc/ntp.conf
~~~

~~~ snip
server 128.178.240.1
~~~

~~~ bash
sudo service ntp restart
~~~


2017-10-02 ● Firewall
------------------------------------------------------------------------

This will set up your server's firewall. To set up EPFL's firewall, you can
contact [ENAC-IT1](mailto:enac-it1@groupes.epfl.ch?Subject=Open%20EPFL%27s%20firewall%20for%20the%20server%20XYZ)

Default policy of a secure firewall config is to deny everything coming to the
server and then allow only the expected protocols. This is what the following
setup does.

For each admin and user's IP who need to ssh, add an incoming rule permission.
Then add other protocols rules (like 80 port for http, 443 for https, ...)

+ Samuel Bancal :
  + 128.178.7.66
+ VPN :
  + 128.178.2.0/24
  + 128.179.252.0/24
  + 128.179.253.0/24
  + 128.179.254.0/24
  + 128.179.255.0/24

~~~ bash
sudo ufw disable
sudo ufw --force reset
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow proto tcp from 128.178.7.66 to any port 22
sudo ufw allow proto tcp from 128.178.2.0/24 to any port 22
sudo ufw allow proto tcp from 128.179.252.0/24 to any port 22
sudo ufw allow proto tcp from 128.179.253.0/24 to any port 22
sudo ufw allow proto tcp from 128.179.254.0/24 to any port 22
sudo ufw allow proto tcp from 128.179.255.0/24 to any port 22
sudo ufw allow proto tcp from any to any port 80
sudo ufw allow proto tcp from any to any port 443
sudo ufw --force enable

sudo ufw status
~~~

~~~ out
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       128.178.7.66              
22/tcp                     ALLOW       128.178.2.0/24            
22/tcp                     ALLOW       128.179.252.0/24          
22/tcp                     ALLOW       128.179.253.0/24          
22/tcp                     ALLOW       128.179.254.0/24          
22/tcp                     ALLOW       128.179.255.0/24          
80/tcp                     ALLOW       Anywhere                  
443/tcp                    ALLOW       Anywhere                  
80/tcp (v6)                ALLOW       Anywhere (v6)             
443/tcp (v6)               ALLOW       Anywhere (v6)             
~~~


2017-10-02 ● Clonezilla setup
------------------------------------------------------------------------

Clonezilla is a great tool to make cold image of your server.
Here is how to set it up so that you can dualboot on it without CD-drive
(or iso to map).

~~~ bash
sudo wget "https://osdn.net/frs/redir.php?m=onet&f=%2Fclonezilla%2F68294%2Fclonezilla-live-20170905-zesty-amd64.iso" -O /clone_sys/clonezilla-live-20170905-zesty-amd64.iso
sudo ln -s clonezilla-live-20170905-zesty-amd64.iso /clone_sys/clonezilla.iso

df -h /clone_sys/

sudo vi /etc/grub.d/40_custom
~~~

~~~ snip
# Note: adapt it to match the partition /clone_sys.
# On my server it's on sda1 which is converted to (hd0,1)
menuentry "Clonezilla live" {
    set root=(hd0,1)
    set isofile="/clonezilla.iso"
    loopback loop $isofile
    linux (loop)/live/vmlinuz boot=live live-config noswap nolocales edd=on nomodeset ocs_live_run=\"ocs-live-general\" ocs_live_extra_param=\"\" ocs_live_keymap=\"\" ocs_live_batch=\"no\" ocs_lang=\"\" vga=788 ip=frommedia nosplash toram=filesystem.squashfs findiso=$isofile
    initrd (loop)/live/initrd.img
}
~~~

~~~ bash
sudo vi /etc/default/grub
~~~

~~~ snip
GRUB_DEFAULT=0
#GRUB_HIDDEN_TIMEOUT=0
GRUB_HIDDEN_TIMEOUT_QUIET=true
GRUB_TIMEOUT=5
~~~

~~~ bash
sudo update-grub2
less /boot/grub/grub.cfg
~~~


2017-10-02 ● Monitored with Icinga2 enacitmon2.epfl.ch
------------------------------------------------------------------------

~~~ bash
sudo vi /etc/apt/sources.list.d/enacrepo.list
~~~

~~~ snip
# http://enacit1.epfl.ch/linux/sys/enacrepo.shtml
deb http://enacrepo.epfl.ch/public xenial main    # pour Ubuntu 16.04 LTS
~~~

~~~ bash
wget -q http://enacrepo.epfl.ch/enacrepo.asc -O- | sudo apt-key add -
sudo apt update && sudo apt install enac-monitoring
~~~

Follow the dedicated documentation
[http://enacit.epfl.ch/monitoring/activation/](http://enacit.epfl.ch/monitoring/activation/)
to enable and have access to the monitoring with ENAC-IT.


2017-10-02 ● ORCA SETUP
------------------------------------------------------------------------

~~~ bash
sudo tar -xf orca_3_0_3_linux_x86-64.tbz -C /opt/
sudo vi /etc/bash.bashrc
~~~

~~~ snip
# ORCA related
export PATH=$PATH:/opt/orca_3_0_3_linux_x86-64
~~~

2017-10-02 ● NBO SETUP
------------------------------------------------------------------------

~~~ bash
sudo tar -xf nbo6.0-bin-linux-x86_64.tar.gz -C /opt/
sudo vi /etc/bash.bashrc
~~~

~~~ snip
# NBO related
NBOP=/opt/nbo6/bin
export NBOVERSION=NBO6
export GENEXE=$NBOP/gennbo.i8.exe
export NBOEXE=$NBOP/nbo6.i8.exe
export PATH=$PATH:$NBOP
~~~


2017-10-02 ● ETH requirements
------------------------------------------------------------------------

~~~ bash
sudo apt install graphviz
~~~

~~~ bash
sudo apt install openjdk-8-jre
~~~

~~~ bash
sudo apt install maven
~~~

~~~ bash
sudo apt install apache2
~~~

~~~ bash
sudo apt install mysql-server
~~~

root password set by Minju Lee.


2017-10-02 ● SSL Certificate : REQUEST
------------------------------------------------------------------------

SSL Certificate : new Request

~~~ bash
sudo vi /etc/ssl/cert_ltqevm1_2017-10-02.cnf
~~~

~~~ snip
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn

[ dn ]
C = CH
O = Ecole polytechnique federale de Lausanne (EPFL)
CN = ltqevm1.epfl.ch
~~~

~~~ bash
sudo openssl req -new -nodes -config /etc/ssl/cert_ltqevm1_2017-10-02.cnf -keyout /etc/ssl/private/ltqevm1_2017-10-02.key -out /etc/ssl/cert_ltqevm1_2017-10-02.csr
~~~

# Visit http://rauth.epfl.ch/
# fill the certificate request form and give the CSR output from :

~~~ bash
cat /etc/ssl/cert_ltqevm1_2017-10-02.csr
~~~

~~~ snip
-----BEGIN CERTIFICATE REQUEST-----
MIICpjCCAY4CAQAwYTELMAkGA1UEBhMCQ0gxODA2BgNVBAoML0Vjb2xlIHBvbHl0
ZWNobmlxdWUgZmVkZXJhbGUgZGUgTGF1c2FubmUgKEVQRkwpMRgwFgYDVQQDDA9s
dHFldm0xLmVwZmwuY2gwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDH
w2wKcbsoYj0tFLOB+GrkRwa+p5X5hA7b1QJnRhvRRRIOe/rvidPErWUiIOQtYZ2l
q96d/EONY7iYG4rcdIs1x6063cNAT6FZgkXQ1/Euijm3SqVXdvjPAhGJzvGDvlCn
ByIVqmKajnfVrN/Uw0BxHH0oKY9YM3lY1G5yO2IwteZ6kHNdkTeNoPmVoTgo8A/Z
qqmoGJiuywAlMjbGJ4TE3akhGAlCWlK81k55D1bjsTz47hYpNPGGm9rrkNEzwJyw
1vdkKnDKod2donvwxl+xiCZwyqN+zLDy8TEDwN4pxeea5nMCh0/P0WIS3PcAnK2r
8GITYGrdyUgIDMWzi2vBAgMBAAGgADANBgkqhkiG9w0BAQsFAAOCAQEASX9Z2plG
4Zys1K22rnQavVRTYs2PZpTcaYG4hw8mRbus+i9XiHDP75PcOZXmpLHWCjh+6VtS
+bO8pWrUXvJmRnWECbUoCxcGxbLRbm6tUKkboMac4TISclAzyLuTfqpdcxkvOhw2
wXRjX94Muvb8SoS8o88yx83n3PgUS9Nsdr+HkCGYuLOswpX5L57wE0v5tQoZ4E3V
Lg8m6/4mUrYU3wt+Ni80WybcwihTJIl2lqUZcOoToZSN5y4u8MzoFvVvW3W0BN3z
3dBn9WA93NNxmzey5VkqGcq/jdCG5fsn4OHXl1A4M7CGnJTiVB9HpGkmVumkumwR
cuTjx2LfWJq1LQ==
-----END CERTIFICATE REQUEST-----
~~~




TODO ... ● Backup
------------------------------------------------------------------------

This is not documented here since it depends much on the data hosted on your
server.

If you need help, please contact
[ENAC-IT1](mailto:enac-it1@groupes.epfl.ch?Subject=Backup%20of%20our%20server%20XYZ).
We'll need to know :

* What is the price of the data (we can loose them -> they are unique)
* What kind of data (files, database, ...)
* What size (GB -> TB)
* What destination storage you have (Unit's NAS, NAS3, USB, ...)


SUPPORT
=======

Visit [http://enacit1.epfl.ch/linux/](http://enacit1.epfl.ch/linux/) for further
information and support.
