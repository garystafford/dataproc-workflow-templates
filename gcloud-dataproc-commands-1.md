# Dataproc Commands

##Main Commands

```bash
gcloud components update

export PROJECT=dataproc-demo-224523
export CLUSTER_1=single-node-cluster
export CLUSTER_2=three-node-cluster
export REGION=us-east1
export ZONE=us-east1-b
export NUM_WORKERS=2
export BUCKET_NAME=gs://dataproc-demo-bucket
export MACHINE_TYPE_SMALL=n1-standard-1
export MACHINE_TYPE_LARGE=n1-standard-4

gcloud config set project $PROJECT

time gcloud dataproc clusters create $CLUSTER_1 \
  --region $REGION --subnet default --zone $ZONE \
  --single-node --master-machine-type $MACHINE_TYPE_SMALL \
  --master-boot-disk-size 500 --image-version 1.3-deb9 \
  --project $PROJECT

time gcloud dataproc clusters create $CLUSTER_2 \
    --region $REGION --subnet default --zone $ZONE \
    --master-machine-type $MACHINE_TYPE_LARGE \
    --master-boot-disk-size 500 \
    --num-workers $NUM_WORKERS \
    --worker-machine-type $MACHINE_TYPE_LARGE \
    --worker-boot-disk-size 500 --image-version 1.3-deb9 \
    --project $PROJECT

time yes | gcloud dataproc clusters delete $CLUSTER_1 --region $REGION

time yes | gcloud dataproc clusters delete $CLUSTER_2 --region $REGION

gsutil mb -p $PROJECT -c regional -l $REGION $BUCKET_NAME

gsutil cp data/ibrd-statement-of-loans-*.csv $BUCKET_NAME
gsutil cp build/libs/dataprocJavaDemo-1.0-SNAPSHOT.jar $BUCKET_NAME
gsutil cp international_loans_dataproc_large.py $BUCKET_NAME

for i in `seq 1 10`;
do
  time gcloud dataproc jobs submit pyspark \
      $BUCKET_NAME/international_loans_dataproc_large.py \
      --region=$REGION \
      --cluster=$CLUSTER_1 \
      --async
done

time gcloud dataproc jobs submit spark \
    --region=$REGION \
    --cluster=$CLUSTER_1 \
    --class=org.example.dataproc.InternationalLoansAppDataproc \
    --jars=$BUCKET_NAME/dataprocJavaDemo-1.0-SNAPSHOT.jar

time gcloud dataproc jobs submit spark \
    --region=$REGION \
    --cluster=$CLUSTER_1 \
    --class=org.example.dataproc.InternationalLoansAppDataprocLarge \
    --jars=$BUCKET_NAME/dataprocJavaDemo-1.0-SNAPSHOT.jar
```

## Data Cleansing

```bash
export FILE = "ibrd-statement-of-loans-historical-data.csv"
(head -n1 ${FILE} |tr '[A-Z]' '[a-z]'; tail -n -2 ${FILE})>foo.csv

awk 'NR==1{$0=tolower($0)} 1' $FILE > temp_file
print a\': | awk "sub(/[:']/, x)"

sed -i '1!b;s/ /-/' $FILE

DataFrame Row Count: 749741

sed -e '1s/ /-/g' $FILE | awk 'NR==1{$0=tolower($0)} 1' $FILE > temp_file.csv

sed -e '1s/fred/frog/' yourfile

https://spark.apache.org/docs/latest/submitting-applications.html#master-urls

https://github.com/apache/spark/blob/master/examples/src/main/java/org/apache/spark/examples/sql/JavaSQLDataSourceExample.java
```
