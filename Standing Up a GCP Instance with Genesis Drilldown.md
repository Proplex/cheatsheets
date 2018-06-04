# Standing Up a GCP Instance with Genesis Drilldown

## Setting up GCP
This section will go over creating and initializing a GCP project.

### Create a GCP Project

And wait. 

### Enabling GCP APIs
A few APIs are necessary to get started. Using the search feature in the blue navigation bar, search for (and enable) the following APIs
		* Cloud Resource Manager API
		* Identity and Access Management API
		* Compute Engine API
		* Google Cloud SQL API
	
### Raise GCP Quotas
Go to the menu (☰), and navigate to “IAM & Admin” -> “Quotas”.  Increase the following:
		* Global In-use IP addresses
			* Set to 50
		* Global CPUs 
			* Set to 100
		* Regional CPUs (the region you’re deploying in)
			* Set to 100

## Setting up Terraform
1. Clone the repository `git clone https://github.com/starkandwayne/codex`
2. Move to the Google Terraform `cd codex/terraform/google`
3. Initialize Terraform `terraform init`
4. Grab credentials from GCP with GCC (the >_ icon in the Cloud Console)
```
export project_id=$(gcloud config get-value project)
export region=us-east1
export zone=us-east1-d
export service_account_email=terraform@${project_id}.iam.gserviceaccount.com

gcloud config set compute/zone ${zone}
gcloud config set compute/region ${region}

gcloud iam service-accounts create terraform --display-name terraform
gcloud iam service-accounts keys create ~/terraform.key.json \
  --iam-account ${service_account_email}

gcloud projects add-iam-policy-binding ${project_id} \
  --member serviceAccount:${service_account_email} \
  --role roles/owner
```
5. Download `terraform.key.json` using the GCP GCC interface, and place it within `keys/` as `iam.json`

**⚠️ Keep  iam.json secure! It has ownership-level access to your GCP project, which grants permission to do anything (including total deletion & access to billing information) to your project ⚠️**

6. Begin populating `google.tfvars` 
	1. Make sure the two prefixes are globally unique, and that they’re all lower-case a-z (no numbers!)
7. Generate SSH keys for the bastion host:
```
ssh-keygen -f keys/gce </dev/null
chmod 0400 keys/*
echo "/keys" >> .gitignore
```

**⚠️ Keep  the generated SSH key secure. This key has total access to your bastion host, and by proxy, all of your GCP infrastructure. ⚠️**