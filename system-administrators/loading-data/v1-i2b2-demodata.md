# v1 \(I2B2 Demodata\)

The _v1_ loader expects an already existing i2b2 database \(in _.csv_ format\) that will be converted in a way that is compliant with the MedCo data model. This involves encrypting and ‘deterministically tagging’ some of the data.

List of input \(‘original’\) files:

> * all i2b2metadata files \(e.g. i2b2.csv\)
> * dummy\_to\_patient.csv
> * patient\_dimension.csv
> * visit\_dimension.csv
> * concept\_dimension.csv
> * modifier\_dimension.csv
> * observation\_fact.csv
> * table\_access.csv

### Dummy Generation

The provided example data set files come with dummy data pre-generated. Those data are random dummy entries whose purpose is to prevent frequency attacks. For more information on how this dummy generation is done please refer to `~/medco-loader/data/scripts/import-tool/report/report.pdf`. In a future release, the generation will be done dynamically by the loader.

### Example

The following example allows to load data into a running MedCo development deployment \(_dev-local-3nodes_\), on the node 0. Adapt accordingly the docker-compose service being ran to load the two other nodes of this profile.

```text
$ cd ~/medco-deployment/compose-profiles/dev-local-3nodes
$ docker-compose -f docker-compose.tools.yml run medco-loader-srv0 v1 \
    --sen /data/i2b2/sensitive.txt \
    --files /data/i2b2/files.toml
```

Explanation of the arguments:

```text
NAME:
    medco-loader v1 - Convert existing i2b2 data model

USAGE:
    medco-loader v1 [command options] [arguments...]

OPTIONS:
    --group value, -g value               UnLynx group definition file
    --entryPointIdx value, --entry value  Index (relative to the group definition file) of the collective authority server to load the data
    --sensitive value, --sen value        File containing a list of sensitive concepts
    --dbHost value, --dbH value           Database hostname
    --dbPort value, --dbP value           Database port (default: 0)
    --dbName value, --dbN value           Database name
    --dbUser value, --dbU value           Database user
    --dbPassword value, --dbPw value      Database password
    --files value, -f value               Configuration toml with the path of the all the necessary i2b2 files
    --empty, -e                           Empty patient and visit dimension tables (y/n)
```

To check that it is working you can query for:

`-> Diagnoses -> Neoplasm -> Benign neoplasm -> Benign neoplasm of breast`

You should obtain 2 matching subjects.

