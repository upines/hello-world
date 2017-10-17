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

It should be noted that the functionalities of some extension modules above will not be available to a user because it requires additional installation of in-house and licensed softwares and are not planned yet to be released to the public. The input excel files, which contain data for numeorus micropollutants (>600) pre-generated using such modules, will be provided instead.   

Although ozonation is currently only available as an appropriate prediction method has been developed by the author, it is of course possible to extend modules for other treatment processes in the future when appropriate methods are available.  

Detailed descriptions on each module follow below. Note that all the options and relevant information for the Python modules described above can be found with -h option, e.g.,
~~~ bash
db_micropols.py -h
~~~

## B. Basic package

The Python modules in this basic package conduct updates of various treatment information such as wastewater treatment plant info, treatment conditions, micropollutant occurrence and abatement data, etc into the database and their retrievals. It is capable of retrieving data in a customized fashion (e.g., extract all the removals of a specific group (antibiotics) of micropollutants for ozonation over all the campaigns in the database), which can be used for further analyses/applications.  

### B.1. Database creation

**Module and relevant input files and tables**  

| Python module | Input file      | Database tables |
|:-------------:|:---------------:|:---------------:|
| db_creation.py| None            | None            |


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
+ **campaigns**:  

   * short reference (first author and year)
   * full reference
   * year
   * URL for the reference


+ **wwtp**:  

   * city (e.g., Lausanne)
   * country (e.g. Switzerland)
   * wastewater type (e.g., municipal, industrial)
   * discharge (m<sup>3</sup>/d)
   * coordinates (longitude, latitude)

**● Treatment process-relevant tables**

+ **biological**: treatment information about biological treatment  
   * type (e.g., activated sludge, MBR)
   * scale (e.g., lab, pilot, full)
   * date/period(YYYY-MM-DD/YYYY-MM-DD-YYYY-MM-DD)
   * solid retention time (day)
   * hydraulic retention time (hours)
   * flow (m<sup>3</sup>/d)
   * miscellaneous info  


+ **ozonation**: treatment information about ozonation

   * injection type (e.g., gas, liquid)
   * scale (lab, pilot, full)
   * date/period (YYYY-MM-DD/YYYY-MM-DD-YYYY-MM-DD)
   * dose (mg/L)
   * dose (gO<sub>3</sub>/gDOC)
   * flow (m<sup>3</sup>/d)
   * sandFilter_included (True/False)
   * miscellaneous info  


   \*`sandFilter_included` is to check whether treatment data such as removals or concentrations corresponding to the ozonation information updated in this table includes the results of both ozonation and the post-sand filtration or not. It is because while some studies report only the treatment data including both treatments, some studies report separately, in which case the treatment data for post-sand filtration is to be updated in the `sandFiltration` table below.

+ **sandFiltration**: treatment information about post sand filtration following ozonation  

   * scale (lab, pilot, full)
   * date/period (YYYY-MM-DD/YYYY-MM-DD-YYYY-MM-DD)
   * flow (m<sup>3</sup>/d)
   * miscellaneous info  


+ **granularAC**: treatment information about granular activated carbon  

   * GAC product info (e.g., CARBOPAL®, Dounau Carbon, Germany)
   * scale (lab, pilot, full)
   * date/period (YYYY-MM-DD/YYYY-MM-DD-YYYY-MM-DD)
   * bed volume (m<sup>3</sup>/m<sup>3</sup>)
   * flow (m<sup>3</sup>/d)
   * empty bed contact time (min)
   * filter velocity (min/hr)
   * miscellaneous info  


+ **poweredAC**: treatment information about powdered activated carbon  

   * PAC product info (e.g., Norit SAE Super)
   * scale (e.g., lab, pilot, full)
   * date/period (YYYY-MM-DD/YYYY-MM-DD-YYYY-MM-DD)
   * dose (mg/L)
   * dose (gPAC/gDOC)
   * flow (m<sup>3</sup>/d)
   * hydraulic retention time (hours)
   * PAC retention time (day)
   * circulation to biological (True/False)
   * miscellaneous info  

   \*`circulation to biological` is to check whether treatment data such as removals or concentrations was obtained from the PAC treatment process with its circulation to the biological treatment or not.


+ **waterquality**: water quality information for each treatment process  

   * water type(influent/effluent of the treatment process)
   * pH
   * temperature(celcius)
   * DOC (mgC/L)
   * alkalinity (mg/LasHCO<sub>3</sub>)
   * total nitrogen (mgN/L)
   * ammonia (mgN/L)
   * nitrate (mgN/L)
   * nitrite (mgN/L)
   * bromide (ug/L)
   * UV<sub>254nm</sub>  


 **● Treatment data-relevant tables**
 + **concentrations**: treatment data about concentrations of micropollutants   

   * influent (ug/L)
   * influent error (ug/L) as standard deviation
   * effluent (ug/L)
   * effluent error (ug/L) as standard deviation   

   \* Note that influent/effluent concentrations can be given not only as a definite value but also as a range such as <10, >3000


 + **removals**: treatment data about removals of micropollutants   

   * removal (%)
   * removal error (%) as standard deviation

   \* Note that removals can be given not only as a definite value but also as a range such as <10, >95

**● Micropollutants-relevant tables**

+ **micropols**: information about micropollutants

   * compound name
   * abbreviation
   * SMILES string
   * CAS number
   * substance group (e.g., pharmaceutical, pesticide, etc)
   * substance subgroup (e.g., antibiotic, fungicide)
   * classification (e.g., transformation products)
   * parent compound in case the micropollutant is a transformation product
   * kOH (second-order rate constant for the reaction of a micropollutant with hydroxyl radicals)
   * reference information (i.e., kOH_ref and kOH_URL) for the kOH value  

   \* kOH, kOH_ref, kOH_URL are the kinetic information for the reaction of a micropollutant with hydroxyl radicals which are additionally given.  


+ **microspecies**: information about the microspecies of micropollutants  

   * SMILES string
   * net charge
   * predicted pKa (acid dissociation constant)
   * predicted tautomeric fraction
   * predicted ko3 (second-order rate constant for the reaction of a micropollutant with ozone)
   * experimental pKa
   * reference information (i.e., pKa_exp_ref and pKa_exp_URL) for experimental pKa
   * experimental tautomeric fraction
   * experimental ko3
   * reference information (i.e., ko3_exp_ref and ko3_exp_URL) for experimental ko3


+ **ko3_site**: information about the predicted site-specific ozone rate constants (ko3_site_pred) for the microspecies of a micropollutant.    

   * site (e.g., phenol, amine, etc.)
   * atoms (e.g., 33,34,35,36,37,38 for phenol, 5 for amine)
   * ko3_site_pred  


### B.2. Mircopollutant updates

**Module and relevant input files and tables**  

| Python module | Input file      | Database tables |
|:-------------:|:---------------:|:---------------:|
| db_micropol.py| micropols.xlsx  | micropols       |


Update a micropollutant table (`micropols`) with the information found in an input file.
By default, it reads in the information in `micropols` worksheet in the `micropols.xlsx` file.

~~~ bash
python db_micropols.py -d dbname -i micropols.xlsx
~~~

`micropols.xlsx` currently contains more than 600 micropollutants relevant to the aquatic environment and their basic information such as name, SMILES, CAS number, classifications, etc, updated by the author as much as possible.
Note that the `micropols` table in a database has fields for kinetic information for the reaction of a micropollutant with hydroxyl radicals such as `kOH`, `kOH_ref`, `kOH_URL`. These fields are to be dealt with using `o3_microspecies.py` module (see below).

### B.3. Campaign data updates (db_campaigns.py)

**Module and relevant input files and tables**  

| Python module  | Input file               | Database tables                                               |
|:--------------:|:------------------------:|:-------------------------------------------------------------:|
| db_campaigns.py| micropols_campaigns.xlsx | campaigns, wwtp, biological, ozonation, granularAC, powderedAC, sandFiltration, waterquality, removals, concentrations|

Update the following tables with the information contained in the worksheet of the same name in the `micropol_campaigns.xlsx` file  

   * campaign-relevant tables such as `campaigns`, `wwtp`
   * treatment process-relevant tables such as `biological`, `ozonation`, `granularAC`, `powderedAC`, and `sandFiltration`, `waterquality`
   * treatment data-relevant tables such as `removals` and `concentrations`

~~~ bash
python db_campaigns.py -d dbname -i micropol_campaigns.xlsx -t table
~~~

Note that in order to update treatment data-relevant tables such as `removals` and `concentrations`, -r option is used instead and the campaign_key needs to be provided as an argument as follows.

~~~ bash
python db_campaigns.py -d dbname -i micropol_campaigns.xlsx -r campaign_key
~~~

Any campaign- and treatment process-relevant tables in the database can be entirely exported in a csv file with tab ('\t') as delimiter as follows.

~~~ bash
python db_campaigns.py -d dbname -e -t table -o filename
~~~

### B.4. Data fetch from database (db_datafetch.py)

**Module and relevant input files and tables**  

| Python module  | Input file               | Database tables  |
|:--------------:|:------------------------:|:----------------:|
| db_datafetch.py| None                     | All              |

Fetch a desired set of data from the database across all the tables mentioned above, i.e., campaign-related tables and micropol table. As shown below, there are many options required in order to fetch the customized dataset. Use -h option to find the detailed instructions for each option.

~~~ bash
python db_datafetch.py -d dbname -t treatment -c campaign_key -i data_key -q waterquality -w wwtp -r result -o filename
~~~

## C. Extension package
In interpreting treatment data for micropollutants during wastewater treatment processes updated using the basic package modules above, treatment process-relevant physico-chemical or kinetic information for micropollutants may be useful.

### C.1. Information updates about microspecies of a micropollutant

**Module and relevant input files and tables**  

| Python module      | Input file                   | Database tables |
|:------------------:|:----------------------------:|:---------------:|
| db_microspecies.py | micropols_microspecies.xlsx, | micropols, microspecies       |


 microspecies are referred to as a group of chemical species for a micropollutant in differing ionization states depending on its ionizing functional groups. In the current version, the `microspecies` table was made to contain the following properties which are mainly intended to be utilized for the ozonation process.  

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
