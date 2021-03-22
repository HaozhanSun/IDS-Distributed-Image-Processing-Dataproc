# Distrbuted Image Processing on Dataproc

This project uses **Apache Spark** on **Google Cloud Dataproc** controlled by **Google Cloud Compute Engine** to distribute a computationally intensive image processing task (stored in **Google Cloud Storage**) onto a cluster of machines. 

## Workflow

1. Create a development machine in Compute Engine.
2. Build a JAR file via ```Scala``` and ```sbt```(to later send to Dataproc).
3. Create a Cloud Storage bucket and collect images.
4. Create a Dataproc cluster.
5. Submit the Spark face detection job to Dataproc.
6. All done and inspect the result.


