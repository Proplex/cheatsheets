# GCP Cheatsheet
#work #cheatsheet #gcp

## To Get Network Info for a VM
This doesn’t always show up for some GCP projects. Not sure why just yet. 

![](GCP%20Cheatsheet/Screen%20Shot%202018-05-23%20at%209.46.51%20AM.png)

## To Create a Key/Secret Pair for Google Cloud Storage (Buckets)
Currently there is no known way to automate this via Terraform. These HMAC credentials are per-user and required for Genesis to deploy a GCP-based CF instance with a Google Storage Bucket.

Go to Storage > Settings > Interoperability. Enable API access, and then generate a key/secret.

![](GCP%20Cheatsheet/Screen%20Shot%202018-05-24%20at%2011.35.03%20AM.png)

## Get User Credentials for PostgreSQL

