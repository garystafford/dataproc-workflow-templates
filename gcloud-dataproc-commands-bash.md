```bash
# good practice to keep the cloud sdk up-to-date
gcloud components update

# login and configure settings
gcloud init

# Your variables go here
export PROJECT_ID=dataproc-demo-224523
export REGION=us-east1
export ZONE=us-east1-b
export BUCKET_NAME=gs://dataproc-demo-bucket

export TEMPLATE_ID=template-demo
gcloud dataproc workflow-templates create \
  $TEMPLATE_ID --region $REGION

gcloud dataproc workflow-templates set-managed-cluster \
  $TEMPLATE_ID \
  --region $REGION \
  --zone $ZONE \
  --cluster-name single-node-cluster \
  --single-node \
  --master-machine-type n1-standard-1 \
  --master-boot-disk-size 500 \
  --image-version 1.3-deb9

gcloud dataproc workflow-templates set-managed-cluster \
  $TEMPLATE_ID \
  --region $REGION \
  --zone $ZONE \
  --cluster-name three-node-cluster \
  --master-machine-type n1-standard-4 \
  --master-boot-disk-size 500 \
  --worker-machine-type n1-standard-4 \
  --worker-boot-disk-size 500 \
  --num-workers 2 \
  --image-version 1.3-deb9

export STEP_ID=ibrd-large-pyspark
gcloud dataproc workflow-templates add-job pyspark \
  $BUCKET_NAME/international_loans_dataproc.py \
  --step-id $STEP_ID \
  --workflow-template $TEMPLATE_ID \
  --region $REGION \
  -- "gs://dataproc-demo-bucket" \
     "ibrd-statement-of-loans-historical-data.csv" \
     "ibrd-summary-large-python"

export STEP_ID=ibrd-small-spark
gcloud dataproc workflow-templates add-job spark \
  --region $REGION \
  --step-id $STEP_ID \
  --workflow-template $TEMPLATE_ID \
  --class org.example.dataproc.InternationalLoansAppDataproc \
  --jars $BUCKET_NAME/dataprocJavaDemo-1.0-SNAPSHOT.jar

export STEP_ID=ibrd-large-spark
gcloud dataproc workflow-templates add-job spark \
  --region $REGION \
  --step-id $STEP_ID \
  --workflow-template $TEMPLATE_ID \
  --class org.example.dataproc.InternationalLoansAppDataprocLarge \
  --jars $BUCKET_NAME/dataprocJavaDemo-1.0-SNAPSHOT.jar

yes | gcloud dataproc workflow-templates remove-job \
  $TEMPLATE_ID \
  --region $REGION \
  --step-id $STEP_ID

gcloud dataproc workflow-templates instantiate \
  $TEMPLATE_ID --region $REGION --async

https://github.com/google/re2/wiki/Syntax

gcloud dataproc workflow-templates import $TEMPLATE_ID \
   --region $REGION --source template-demo-param.yaml

time gcloud dataproc workflow-templates instantiate \
  $TEMPLATE_ID --region $REGION --async \
  --parameters MAIN_PYTHON_FILE="gs://dataproc-demo-bucket/international_loans_dataproc.py",STORAGE_BUCKET="gs://dataproc-demo-bucket",IBRD_DATA_FILE="ibrd-statement-of-loans-historical-data.csv",RESULTS_DIRECTORY="ibrd-summary-large-python"

time gcloud dataproc workflow-templates instantiate \
  $TEMPLATE_ID --region $REGION --async \
  --parameters MAIN_PYTHON_FILE="gs://dataproc-demo-bucket/international_loans_dataproc.py",STORAGE_BUCKET="gs://dataproc-demo-bucket",IBRD_DATA_FILE="ibrd-statement-of-loans-latest-available-snapshot.csv",RESULTS_DIRECTORY="ibrd-summary-small-python"

gcloud dataproc workflow-templates list --region $REGION

gcloud dataproc workflow-templates describe \
  $TEMPLATE_ID --region $REGION

gcloud dataproc operations list --region $REGION

gcloud dataproc operations describe \
  projects/$PROJECT_ID/regions/$REGION/operations/c1d89aa5-c276-3a55-b46d-30592c8e59fe

gcloud dataproc operations cancel ea993765-edba-4db5-8536-a864227408d7

yes | gcloud dataproc workflow-templates delete \
  $TEMPLATE_ID --region $REGION

gcloud dataproc workflow-templates instantiate-from-file \
  --file template-demo.yaml \
  --region $REGION \
  --async
```
