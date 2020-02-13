# v0 \(Genomic Data\)

The _v0_ loader expects an ontology, with mutation and clinical data in the MAF format. As the ontology data you must use `~/medco-deployment/resources/data/genomic/tcga_cbio/clinical_data.csv` and `~/medco-deployment/resources/data/genomic/tcga_cbio/mutation_data.csv`. For clinical data you can keep using the same two files or a subset of the data \(e.g. 8\_clinical\_data.csv\). More information about how to generate sample datafiles can be found below. After the following script is executed all the data is encrypted and ‘deterministically tagged’ in compliance with the MedCo data model.

### Example

The following example allows to load data into a running MedCo development deployment \(_dev-local-3nodes_\), on the node 0. Adapt accordingly the docker-compose service being ran to load the two other nodes of this profile.

```text
$ cd ~/medco-deployment/compose-profiles/dev-local-3nodes
$ docker-compose -f docker-compose.tools.yml run medco-loader-srv0 v0 \
    --ont_clinical /data/genomic/tcga_cbio/8_clinical_data.csv \
    --sen /data/genomic/sensitive.txt \
    --ont_genomic /data/genomic/tcga_cbio/8_mutation_data.csv \
    --clinical /data/genomic/tcga_cbio/8_clinical_data.csv \
    --genomic /data/genomic/tcga_cbio/8_mutation_data.csv \
    --output /data/
```

Explanation of the arguments:

```text
NAME:
    medco-loader v0 - Load genomic data (e.g. tcga_bio dataset)

USAGE:
    medco-loader v0 [command options] [arguments...]

OPTIONS:
    --group value, -g value               UnLynx group definition file
    --entryPointIdx value, --entry value  Index (relative to the group definition file) of the collective authority server to load the data
    --sensitive value, --sen value        File containing a list of sensitive concepts
    --dbHost value, --dbH value           Database hostname
    --dbPort value, --dbP value           Database port (default: 0)
    --dbName value, --dbN value           Database name
    --dbUser value, --dbU value           Database user
    --dbPassword value, --dbPw value      Database password
    --ont_clinical value, --oc value      Clinical ontology to load
    --ont_genomic value, --og value       Genomic ontology to load
    --clinical value, --cl value          Clinical file to load
    --genomic value, --gen value          Genomic file to load
    --output value, -o value              Output path to the .csv files
```

### Data Manipulation

Inside `~/medco-loader/data/scripts/` you can find a small python application to extract \(or replicate\) data out of the original _tcga\_cbio_ dataset. You can decide which patients you want to consider for you ‘new’ dataset or simply randomly pick a sample.

To check that it is working you can query for:

`-> MedCo Gemomic Ontology -> Gene Name -> BRPF3`

For the small dataset `8_xxxx` you should obtain 3 matching subjects \(one at each site\).

