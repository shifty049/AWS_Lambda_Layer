# AWS Lambda Layer

## Takeaways during developing AWS lambda layer

## Reference to [Here!](https://medium.com/@qtangs/creating-new-aws-lambda-layer-for-python-pandas-library-348b126e9f3e)

## `Size limit:` 
1. each zip file not greater than **50 MB** 
2. sum of all layers used in AWS lambda not greater than **250 MB** (**`* unzipped`**)
3. each Lambda function can use at most **5** layers

## `challenge` - libraries with big volume for being included in layer's 250 MB limit
1. mysql-connector-python
2. pandas
3. pyarrow


# create an zip file of AWS layer well fitting the environment of AWS Lambda 

## Bash script for creating AWS layer file

> **`get_layer_packages.sh`**
- use docker to install necessary libraries for fitting the environment of AWS Lambda
- use requirements_deps ti install libraries without dependent libraries
- -se requirements_deps ti install libraries with their dependency

```
#!/bin/bash

export PKG_DIR="python"

rm -rf ${PKG_DIR} && mkdir -p ${PKG_DIR}
docker run --rm -v $(pwd):/foo -w /foo lambci/lambda:build-python3.7 \
    pip install -r requirements_deps.txt --no-deps -t ${PKG_DIR} &&\ 
docker run --rm -v $(pwd):/foo -w /foo lambci/lambda:build-python3.7 \
    pip install -r requirements.txt -t ${PKG_DIR}
```

## Bash script for synchronizing my self-established module to the AWS layer file  

> **`module_sync.sh`**
- use rsync to sync modules created on my own and key file to AWS Lambda directory
```
#!/bin/bash

# rsync module & key use in AWS lambda layer
original_key_path=/Users/data/lambda_key/
alternative_keymanagement_path=/Users/benq/Desktop/necessary_package/python/Key_Management
original_module_path=/Users/benq/Documents/Project/github/data_etl/sub_functions/
module_path=/Users/benq/Desktop/necessary_package/python/python 
rsync --delete -avr $original_key_path $module_path"/.keys" &&
rsync --delete -avr $original_module_path"Slack" $module_path &&
rsync --delete -avr $original_module_path"GCP_GoogleSheet" $module_path &&
rsync --delete -avr $alternative_keymanagement_path $module_path &&
rsync --delete -avr $original_module_path"GCP_BigQuery" $module_path &&
rsync --delete -avr $original_module_path"MySQLHandler" $module_path
```
> **`lambda_layer_creating.sh`**
- integrate two parts mentioned above and compress the directory into a zip file
```
#!/bin/bash

default_zip_file_name="python37_layer.zip"
zip_file_name=${1:-$default_zip_file_name}
cd /Users/benq/Desktop/necessary_package/python &&\ 
rm "./"$zip_file_name &&\ 
./get_layer_packages.sh && ./module_sync.sh &&\ 
zip -r9 "./"$zip_file_name ./python
```

- path of uploaded python layer on AWS Lambda: `/opt/python`
- path of executed lambda function on AWS Lambda: `/var/task`

