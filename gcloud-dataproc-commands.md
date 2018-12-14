```cmd
# Install gcloud sdk on windows behind proxy

gcloud config set proxy/type http
gcloud config set proxy/address localhost
gcloud config set proxy/port 3128

gcloud config unset proxy/type
gcloud config unset proxy/address
gcloud config unset proxy/port

gcloud components update

# login and configure settings
gcloud init

# Create and run workflow

set PROJECT_ID=dataproc-demo-224523
set REGION=us-east1
set ZONE=us-east1-b
set TEMPLATE_ID=template-demo
set CLUSTER_1=single-node-cluster
set CLUSTER_2=three-node-cluster
set MACHINE_TYPE_SMALL=n1-standard-1
set MACHINE_TYPE_LARGE=n1-standard-4
set NUM_WORKERS=2
set BUCKET_NAME=gs://dataproc-demo-bucket
set STEP_ID=top-ibrd-borrowers

gcloud dataproc workflow-templates create ^
  %TEMPLATE_ID% --region %REGION%

gcloud dataproc workflow-templates set-managed-cluster ^
  %TEMPLATE_ID% ^
  --region %REGION% ^
  --cluster-name %CLUSTER_1% ^
  --zone %ZONE% ^
  --single-node ^
  --master-machine-type %MACHINE_TYPE_SMALL% ^
  --master-boot-disk-size 500 ^
  --image-version 1.3-deb9

gcloud dataproc workflow-templates set-managed-cluster ^
  %TEMPLATE_ID% ^
  --region %REGION% ^
  --cluster-name %CLUSTER_2% ^
  --zone %ZONE% ^
  --master-machine-type %MACHINE_TYPE_LARGE% ^
  --master-boot-disk-size 500 ^
  --num-workers %NUM_WORKERS% ^
  --worker-machine-type %MACHINE_TYPE_LARGE% ^
  --worker-boot-disk-size 500 ^
  --image-version 1.3-deb9

gcloud dataproc workflow-templates add-job pyspark ^
  %BUCKET_NAME%/international_loans_dataproc_large.py ^
  --step-id %STEP_ID% ^
  --workflow-template %TEMPLATE_ID% ^
  --region %REGION%

set STEP_ID=top-ibrd-borrowers-small

gcloud dataproc workflow-templates add-job spark ^
  --region %REGION% ^
  --step-id %STEP_ID% ^
  --workflow-template %TEMPLATE_ID% ^
  --class org.example.dataproc.InternationalLoansAppDataproc ^
  --jars %BUCKET_NAME%/dataprocJavaDemo-1.0-SNAPSHOT.jar

set STEP_ID=top-ibrd-borrowers-large

gcloud dataproc workflow-templates add-job spark ^
  --region %REGION% ^
  --step-id %STEP_ID% ^
  --workflow-template %TEMPLATE_ID% ^
  --class org.example.dataproc.InternationalLoansAppDataprocLarge ^
  --jars %BUCKET_NAME%/dataprocJavaDemo-1.0-SNAPSHOT.jar

echo y | gcloud dataproc workflow-templates remove-job ^
  %TEMPLATE_ID% ^
  --region %REGION% ^
  --step-id %STEP_ID%

gcloud dataproc workflow-templates instantiate ^
  %TEMPLATE_ID% --region %REGION% --async

gcloud dataproc workflow-templates list --region %REGION%

gcloud dataproc workflow-templates describe ^
  %TEMPLATE_ID% --region %REGION%

gcloud dataproc operations list --region %REGION%

gcloud dataproc operations describe ^
  projects/%PROJECT_ID%/regions/%REGION%/operations/c1d89aa5-c276-3a55-b46d-30592c8e59fe

gcloud dataproc workflow-templates instantiate-from-file ^
  --file template-demo.yaml ^
  --region %REGION% ^
  --async

```
