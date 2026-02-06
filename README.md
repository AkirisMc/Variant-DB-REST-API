# Variant-DB-REST-API
This repository provides the technical infrastructure for establishing a relational database optimised for storing variant data, along with a REST API service that enables convenient and programmatic access to the data.

## Database
This technical infrastructure leverages the SQLite engine (v3.34.1) to establish a relational database designed for storing variant data extracted from VCF files (v4.1).

The database schema, defined in [schema.sql](https://github.com/AkirisMc/Variant-DB-REST-API/blob/main/Database/schema.sql), contains the SQL code required to create the database structure. The schema was originally designed to accommodate variant data from different *Arabidopsis thaliana* strains; however, it can be adapted to store variant data from other organisms. The database follows a straightforward and modular design to facilitate both data population and efficient querying.

The example database, [database.sqlite](https://github.com/AkirisMc/Variant-DB-REST-API/blob/main/Database/database.sqlite), is a populated instance created using this infrastructure. The steps followed to design, populate, and query the database are described in the sections below.

<p align="center">
<img src="https://github.com/AkirisMc/Variant-DB-REST-API/blob/main/Images/Database_ER_diagram.jpg" width="450">
</p>

The structure of the database is made up of two tables, the *Datasets* table and the *Variants* table. 

The *Datasets* table stores unique genome identifiers (ideally reflected in each file name e.g. *7208*) under the field **genome**, along with the corresponding full file names (e.g. *genome_7208.vcf*) stored under **dataset**.

| **genome**      | **dataset**                   |
| :-------------: | :---------------------------: |
| 7208            | genome_7208.vcf               |
| 8233            | genome_8233.vcf               |
| 9968            | genome_9968.vcf               |

The *Variants* table contains eight attributes which represent the information present in the columns of a VCF file. 

To streamline the data and retain essential information, the values typically found in the *ID*, *FILTER*, and *FORMAT* columns were omitted, including only the genotype data (e.g. *1|1*) from the final column of the VCF files.

An identifying attribute, designated as **id**, was introduced as an incremental integer to uniquely identify each row.

| **id**     | **genome**   | **chromosome**   | **position**  | **reference**   | **alternate** | **quality**   | **read depth** | **genotype**  | 
| :----: | :------: | :--------:   | :-------: | :-------:   | :-------: | :-------: | :--------: | :-------: |
| 1      | 7208     | 1            | 6324      | T           | TA        | 40        | 104        | 1|1       |
| 2      | 7208     | 1            | 214644    | A           | AT        | 40        | 39         | 1|1       |
| 3      | 7208     | 1            | 851442    | T           | TA        | 40        | 135        | 1|1       |

## Importing the data
The database was populated using the Python script [dataimport_tool.py](https://github.com/AkirisMc/Variant-DB-REST-API/blob/main/dataimport_tool.py). The program takes the names of VCF files as command-line arguments, parsing each file and populating the database tables accordingly. It is designed to handle a variable number of VCF files.

To run the program, users just need to enter the name of the VCF files whose content they want to be uploaded to the database.

```python dataimport_tool.py file1.vcf file2_name.vcf file3_name.vcf```  

**Note:** For successful execution of the program, ensure that the script, the VCF files, and the database file are located in the same directory. 

## Server 
A RESTful web service was created using the *Node.js* environment and the *Express.js* framework to enable users to programmatically retrieve and interact with data stored in the variant database.

In the [server.js](https://github.com/AkirisMc/Variant-DB-REST-API/blob/main/Server/server.js) script the *Express* dependency is loaded, an object is instantiated using the ```express()``` method, and subsequently, an HTTP server is launched to listen for requests on ```port 3000```. 

A router ([router.js](https://github.com/AkirisMc/Variant-DB-REST-API/blob/main/Server/router.js)) was created to organise and modularise the handling of different API endpoints, which was later imported into *server.js* and linked to the API path.

## Server deployment 
Users need to deploy the server first in order to interact with the database. To deploy the server, the content of this repository needs to be cloned on your device. 
```
git clone https://github.com/AkirisMc/Variant-DB-REST-API.git
cd Variant-DB-REST-API.git
```
Users then should navigate to the **Server** directory, which includes the *server.js* file along with other associated files. To deploy the server, execute the *server.js* file in the terminal.
```
cd Server
node server.js
```
Upon successful deployment, users should observe the following message displayed in the terminal:

```Application deployed on port 3000```

## API endpoints 
To access data from the database, users must utilise the REST API endpoints through URL extensions. Various endpoints were created to enable users the access of different types of data from the variant database.

### Testing 
This example shows how to test the server via URL to check whether it is capable of receiving requests and sending responses. Users just need to type the following URL on their browser:

```http://localhost:3000/api/test```

The following message should be displayed on the browser:

```request was successfully received and response was successfully sent```

Alternatively, the same request can be made using the command line:

```curl http://localhost:3000/api/test```

### Endpoint to report the datasets
This endpoint fetches all the dataset values (i.e. the name of the VCF files uploaded to the database) and presents them to the user in the form of an array.

| Endpoint                    | Description                                                         |
| :-------------------------: | :-----------------------------------------------------------------: |
| ```/api/datasets```         | Displays the name of the VCF files uploaded to the database         |

Example output: ```[“genome_7208.vcf”, “genome_8233.vcf”, “genome_9968.vcf”]```

### Endpoints to report the number of variants
These endpoints count the number of structural variants (i.e. SNPs, InDels, or both) per chromosome given the numerical identifier of a genome (e.g. *7208*). The resulting counts are then exhibited for each chromosome in the format of a dictionary.

| Endpoint                                 | Description                                                                  |
| :--------------------------------------: | :--------------------------------------------------------------------------: |
| ```/api/numbervariants/snps/:genome```   | Displays the number of SNPs per chromosome given a genome                    |
| ```/api/numbervariants/indels/:genome``` | Displays the number of InDels per chromosome given a genome                  |
| ```/api/numbervariants/both/:genome```   | Displays the number of both types of variants per chromosome given a genome  |

Example output: ```{“chromosome1”:“21513”,“chromosome2”:“45”,“chromosome3”:“98”,“chromosome4”:“24523”, “chromosome5”:“53”}``` 

### Endpoints to report all the information on specific variants 
These endpoints list all the information on specific variants (i.e. SNPs, InDels, or both), given a numerical genome identifier (e.g. *7208*), a chromosome number (e.g. *1*), and the start and end positions on that chromosome (e.g. *6324 - 95475*). 

In simpler terms, these endpoints can show comprehensive details of all the SNPs, InDels, or both variant types, found within a defined region of a specified chromosome in a designated dataset. The information is displayed in a JSON format.

| Endpoint                                 | Description                                                                  |
| :--------------------------------------: | :--------------------------------------------------------------------------: |
| ```/api/listvariants/snps/:genome/:chrom/:startpos/:endpos```   | Displays a list of SNPs given a genome, a chromosome, a start position and an end position                    |
| ```/api/listvariants/indels/:genome/:chrom/:startpos/:endpos``` | Displays a list of InDels given a genome, a chromosome, a start position and an end position                  |
| ```/api/listvariants/both/:genome/:chrom/:startpos/:endpos```   | Displays a list of both types of variants given a genome, a chromosome, a start position and an end position  |

Example output: 
```
[{"id":1,"genome":7208,"chromosome":1,"position":6324,"reference":"T","alternate":"TA","quality":40,"read_depth":104,"genotype":"1|1"},
{"id":2,"genome":7208,"chromosome":1,"position":214644,"reference":"A","alternate":"AT","quality":40,"read_depth":39,"genotype":"1|1"},
{"id":3,"genome":7208,"chromosome":1,"position":851442,"reference":"T","alternate":"TA","quality":40,"read_depth":135,"genotype":"1|1"},
{"id":4,"genome":7208,"chromosome":1,"position":863646,"reference":"G","alternate":"GA","quality":40,"read_depth":29,"genotype":"1|1"},
{"id":5,"genome":7208,"chromosome":1,"position":869553,"reference":"C","alternate":"CT","quality":40,"read_depth":149,"genotype":"1|1"},
{"id":6,"genome":7208,"chromosome":1,"position":874471,"reference":"G","alternate":"GA","quality":40,"read_depth":84,"genotype":"1|1"},
{"id":7,"genome":7208,"chromosome":1,"position":895475,"reference":"G","alternate":"GA","quality":40,"read_depth":116,"genotype":"1|1"}]
```

### Endpoints to report the average sample quality
These endpoints calculate the average quality of the variants (i.e. SNPs, InDels, or both) per chromosome given the numerical identifier of a genome (e.g. *7208*). The results are then rounded and displayed in the form of a dictionary.

| Endpoint                                 | Description                                                                             |
| :--------------------------------------: | :-------------------------------------------------------------------------------------: |
| ```/api/avquality/snps/:genome```   | Displays the average sample quality of SNPs per chromosome given a genome                    |
| ```/api/avquality/indels/:genome``` | Displays the average sample quality of InDels per chromosome given a genome                  |
| ```/api/avquality/both/:genome```   | Displays the average sample quality of both types of variants per chromosome given a genome  |

Example output: ```{“chromosome1”:“38.32”,“chromosome2”:“37.02”,“chromosome3”:“38.55”,“chromosome4”:“38.34”, “chromosome5”:“37.25”}```
