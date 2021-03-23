# Distrbuted Image Processing on Dataproc

This project uses **Apache Spark** on **Google Cloud Dataproc** controlled by **Google Cloud Compute Engine** to distribute a computationally intensive image processing task (stored in **Google Cloud Storage**) onto a cluster of machines. 


## Workflow

1. Create a development machine in Compute Engine.
2. Build a JAR file via ```Scala``` and ```sbt```(to later send to Dataproc).
3. Create a Cloud Storage bucket and collect images.
4. Create a Dataproc cluster.
5. Submit the Spark face detection job to Dataproc.
6. All done and inspect the result.

### Step 1: Create a development machine in Google Cloud Compute Engine

Go to Compute Engine > VM Instances > Create.

Configure the following fields, leave the others at their default value.

Name: devhost

Series: N1

Machine Type: 2 vCPUs (n1-standard-2 instance)

Identity and API Access: Allow full access to all Cloud APIs.

### Step 2: Install Software

Setup Scala and sbt

```
sudo apt-get install -y dirmngr unzip
sudo apt-get update
sudo apt-get install -y apt-transport-https
echo "deb https://dl.bintray.com/sbt/debian /" | \
sudo tee -a /etc/apt/sources.list.d/sbt.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 642AC823
sudo apt-get update
sudo apt-get install -y bc scala sbt
```

 Setup Feature Detector Files

```
sudo apt-get update
gsutil cp gs://spls/gsp124/cloud-dataproc.zip .
unzip cloud-dataproc.zip
cd cloud-dataproc/codelabs/opencv-haarcascade
```

#### Launch Build

This command builds a "fat JAR" of the Feature Detector so that it can be submitted to Cloud Dataproc.
```
sbt assembly
```

### Step 3: Create a Cloud Storage bucket and collect images

```
GCP_PROJECT=$(gcloud config get-value core/project)
MYBUCKET="${USER//google}-image-${RANDOM}"
echo MYBUCKET=${MYBUCKET}
```
Use the gsutil program to create the bucket to hold our sample images.
```
gsutil mb gs://${MYBUCKET}
```
Then, Download some sample images into our bucket.
```
curl https://www.publicdomainpictures.net/pictures/20000/velka/family-of-three-871290963799xUk.jpg | gsutil cp - gs://${MYBUCKET}/imgs/family-of-three.jpg
curl https://www.publicdomainpictures.net/pictures/10000/velka/african-woman-331287912508yqXc.jpg | gsutil cp - gs://${MYBUCKET}/imgs/african-woman.jpg
curl https://www.publicdomainpictures.net/pictures/10000/velka/296-1246658839vCW7.jpg | gsutil cp - gs://${MYBUCKET}/imgs/classroom.jpg
```
A sample downloaded image:
![image](https://user-images.githubusercontent.com/37522943/112071624-90e79600-8b46-11eb-84a7-7e98f87f54c8.png)

See the contents of our bucket:
```
gsutil ls -R gs://${MYBUCKET}
```

### Step 4: Create a Cloud Dataproc cluster
```
MYCLUSTER="${USER/_/-}-qwiklab"
echo MYCLUSTER=${MYCLUSTER}
```

Set a global Compute Engine region to use and create a new cluster:
```
gcloud config set dataproc/region us-central1
gcloud dataproc clusters create ${MYCLUSTER} --bucket=${MYBUCKET} --worker-machine-type=n1-standard-2 --master-machine-type=n1-standard-2 --initialization-actions=gs://spls/gsp010/install-libgtk.sh --image-version=2.0  
```

### Step 5: Submit the face detection job to Dataproc

```
curl https://raw.githubusercontent.com/opencv/opencv/master/data/haarcascades/haarcascade_frontalface_default.xml | gsutil cp - gs://${MYBUCKET}/haarcascade_frontalface_default.xml
```
Use the set of images uploaded into the imgs directory in Cloud Storage bucket as input to the Feature Detector
```
cd ~/cloud-dataproc/codelabs/opencv-haarcascade
```
Submit the job to Dataproc:
```
gcloud dataproc jobs submit spark \
--cluster ${MYCLUSTER} \
--jar target/scala-2.12/feature_detector-assembly-1.0.jar -- \
gs://${MYBUCKET}/haarcascade_frontalface_default.xml \
gs://${MYBUCKET}/imgs/ \
gs://${MYBUCKET}/out/
```

Then we can monitor the job, in the Console go to Navigation menu > Dataproc > Jobs.

When the job is complete, go to Navigation menu > Storage and find the bucket we created.

click on an image in the Out directory. Click on Download icon, the image will download to local PC.

Sample output:
![image](https://user-images.githubusercontent.com/37522943/112072164-a3160400-8b47-11eb-8a77-57c84fde2441.png)

Then we are all done. We successfully used Spark on Dataproc to process images in cluster machines.
