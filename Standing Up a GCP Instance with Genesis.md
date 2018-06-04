# Standing Up a GCP Instance with Genesis
#work #guide #genesis #gcp

Get you a GCP!

## Initial Setup
First, we’ll create a GCP project and clone some necessary utilities to get started. You’ll want a credit card handy, as GCP isn’t free. You also won’t be able to use a trial GCP project for this, as we require more than one IP in use (among other things) 

Some things to decide before getting started:
		* What region are we going to use to deploy GCP? If you’re unsure or just want to muck around with GCP, use `us-east1`
				* East coast, beast coast

### Creating GCP Project
Visit [Google Cloud Console](https://console.cloud.google.com/home/) and create a new project:
![](Standing%20Up%20a%20GCP%20Instance%20with%20Genesis/Screen%20Shot%202018-05-24%20at%201.10.11%20PM%203.png)
![](Standing%20Up%20a%20GCP%20Instance%20with%20Genesis/Screen%20Shot%202018-05-24%20at%201.10.21%20PM%203.png)
![](Standing%20Up%20a%20GCP%20Instance%20with%20Genesis/Screen%20Shot%202018-05-24%20at%201.10.48%20PM%203.png)

If you don’t have a billing account setup, you’ll be asked to input your information. 

### Enabling GCP APIs
A few APIs are necessary to get started. Using the search feature in the blue navigation bar, search for (and enable) the following APIs:
		* Cloud Resource Manager API
		* Identity and Access Management API
		* Compute Engine API
		* Google Cloud SQL API
![](Standing%20Up%20a%20GCP%20Instance%20with%20Genesis/Screen%20Shot%202018-05-24%20at%201.30.51%20PM%203.png)
![](Standing%20Up%20a%20GCP%20Instance%20with%20Genesis/Screen%20Shot%202018-05-24%20at%201.55.45%20PM%203.png)
	

### Raising GCP Quotas
It’s also necessary to raise some too-low quota values that Google Cloud declares for new projects.  Go to the menu (☰), and navigate to “IAM & Admin” -> “Quotas”.  You’ll need to increase the following:
			* Global In-use IP addresses
				* Set to 50
			* Global CPUs 
				* Set to 100
			* Regional CPUs (the region you’re deploying in)
				* Set to 100
Since there’s a lot of quotas to scroll through, your best bet is to _select none_ under the Metric dropdown, and search for the quotas you need and select them. 

![](Standing%20Up%20a%20GCP%20Instance%20with%20Genesis/Screen%20Shot%202018-05-24%20at%202.41.59%20PM%203.png)

Raising quotas requires Google approval, so it may take some time to the new quotas to be set.  You can continue with the rest of the setup while you wait, but you can’t Terraform GCP (and anything after that) until they’re set. _It can take up to 48 hours, but it’s typically 5-40 minutes_

### Clone Codex Repository
Codex is a repository we (Stark & Wayne) maintain as a knowledge DB. It contains scripts & information that helps get various projects setup (including this one). So, clone it in your typical work directory:

```
git clone https://github.com/starkandwayne/codex
```

For the remainder of this tutorial, we’ll be working in `codex/terraform/google`


### Grabbing Credentials From GCP
Now that your project is setup, you need to grab some keys and information necessary for Terraform to work. Open the Google Cloud Console (the >_ icon from the upper right hand navbar) and run the following Bash commands in the Cloud Console:
![](Standing%20Up%20a%20GCP%20Instance%20with%20Genesis/Screen%20Shot%202018-05-25%20at%2010.02.17%20AM%203.png)

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

You’ve now created a user account for Terraform to use, and granted it owner status. You’ll need to download the `terraform.key.json` file that was just generated, which you can do by clicking on the Console dropdown (⋮), selecting “Download file”, and typing in `terraform.key.json`. Place that file in your Codex repository, as `terraform/google/keys/iam.json`. 

**⚠️ Keep  iam.json secure! It has ownership-level access to your GCP project, which grants permission to do anything (including total deletion & access to billing information) to your project ⚠️**

### Setting Up Terraform

Install Terraform on your system. If you’re on Mac and use Homebrew, it’s `brew install terraform`, otherwise you’ll need to visit [Terraform Downloads](https://www.terraform.io/downloads.html) and download the binary for your system.

Once you have Terraform, `cd` into the Codex repository, and into `terraform/google` and run `terraform init`. This will download the necessary plugins to use the GCP API.

Some of the Terraform files require an external library called cc-me, created by James. You’ll need to install it:
```
wget https://raw.githubusercontent.com/jhunt/cc-me/master/cc-me
chmod +x cc-me
sudo mv cc-me /usr/bin/local/cc-me
```

A file containing per-project variables, named `google.tfvars` needs to be created within `terraform/google` and populated with:
```
google_project      = "<< project id >>"
google_region       = "<< gcp zone >> "
google_az1          = "b"
google_az2          = "c"
google_az3          = "d"
google_network_name = "codex"
google_credentials  = "keys/iam.json"
google_pubkey_file  = "keys/gce.pub"
bucket_prefix = " << random all lower-case string >> "
db_prefix = " << random all lower-case string >> "
```

Here’s an example `tfvars`: 
```
google_project      = "codex-tutorial"
google_region       = "us-east1"
google_az1          = "b"
google_az2          = "c"
google_az3          = "d"
google_network_name = "codex"
google_credentials  = "keys/iam.json"
google_pubkey_file  = "keys/gce.pub"
bucket_prefix = "2007178a2c3148f"
db_prefix = "8b21d03cb5c74ca8b"
```

Now, you’ll need SSH keys to get into the bastion host and the NATs. You can generate a key with these commands (within `terraform/google`)
```
ssh-keygen -f keys/gce </dev/null
chmod 0400 keys/*
echo "/keys" >> .gitignore
```

**⚠️ Keep  the generated SSH key secure. This key has total access to your bastion host, and by proxy, all of your GCP infrastructure. ⚠️**


## Terraform the GCP Project
Now that you’ve gotten all the pre-reqs down, it’s time to Terraform the GCP instance. Included was a `Makefile` to automatically run the necessary commands. So, run `make`  and Terraform will run pre-flight checks, and then prompt you to continue. Type `yes`, and off we go.

The Terraform script does the following:
	* Creates the necessary networks for an ops plane, dev environment, staging environment, and prod environment
	* Creates a dev, staging, prod PostgreSQL instance (for use with CF)
	* Creates a dev, staging, prod Google Storage Buckets (for use with CF)
	* Creates a dev, staging, and prod load balancers (for use with CF)
	* Creates 3 HA NATs  (VM)
	* Creates a bastion host (which acts as the jumpbox)  (VM)

This will take ~15 minutes.  When it’s complete, run `certify` and then `make cloud`. This will generate the cloud configs for all the BOSH environments. We’ll return to those files in a bit.

`certify`  _will segfault, but that's OK since we purposely kill the process mid-job_
- - - -
## Deploying the Ops Plane
The ops plane contains the following:
	* Proto BOSH
	* SHIELD
	* Vault
	* Concourse
	* Jumpbox

This layer is used to manage the three main environments: _dev, staging, prod_

### Deploying the Proto BOSH 
Now, we want to deploy our Proto BOSH, which starts with SSH’ing into the Bastion host. There’s already a script within the `terraform/google` folder named `to-the-bastion` which will automagically grab the IP of the Bastion host from the Terraform state file and SSH you into the box.

You should open two terminals and maintain two SSH connections to the Bastion host. Makes life easy.

First, we’ll need to setup a local Vault that’s used to store secrets generated by BOSH during the deploy. To do so, use one of your SSH sessions and run: `safe --memory --as dev`

We’ll be using [Genesis](https://github.com/starkandwayne/genesis) to standup any BOSH-related projects. So:

Run `genesis init -k bosh` from the home folder. This will create a new Genesis deployment folder, which will be used to track all BOSH deployments within this GCP instance. This folder is called `bosh-deployments` and is also a Git repository, so you can push these files somewhere to keep them safe (among other reasons).

Within the `bosh-deployments` directory, run `genesis new <environment name>`.  (For the record, I used `codex-tutorial`) This will start a configuration wizard asking for the necessary information required to deploy. Here’s what you should input:

**Which vault would you like to target?** `dev`
This is the local Vault we’ve just setup.

**Is this a proto-BOSH director?** `y`

**What static IP do you want to deploy this BOSH director on?** `10.4.1.5`
This IP is chosen due to the Terraform and cloud config generated. It may seem like it’s pulled out of thin air, but if you view the Terraform plans you’ll see that the address range the Ops plane lives on is `10.4.1/24`, and `.5` is the next IP in line (`.1` is the gateway, `.2-.4` are the NATs)

**What network should this BOSH director exist in (in CIDR notation)** `10.4.1.0/24`
See above—this range was chosen through the Terraform.

**What default gateway (IP address) should this BOSH director use?**  `10.4.1.1`
See above :)

**What DNS servers should BOSH use? (leave value empty to end)** `8.8.8.8` and `8.8.4.4`
You can use whatever DNS servers you’d prefer, but it makes sense to use Google DNS in a Google Cloud service.

**What IaaS will this BOSH director orchestrate?** `3` (Google Cloud Platform)
Necessary for what CPIs and configuration values to use.

**What is your GCP project ID?**
This can be found from the `keys/iam.json` file, the value for the key `project_id`

**What are your GCP credentials?**
Paste the contents of `keys/iam.json` into the terminal and press ctrl-D

**What is your GCP network name?** `codex`
This is defined from the Terraform, we’ve created an entire VPC called `codex` that contains all the routing and firewall rules. You can see the details about them either in the GCP console under “VPC network” or through the Terraform plan.

**What is your GCP subnetwork name?** `codex-global-infra`
Global infrastructure is the name used to reference the network that our Ops plane will be deployed on (the 10.4.1.0/24 range)

**What availability zone do you want the BOSH VM to reside in?** `us-east1-b`
That’s the (Google!) name of the availability zone. If you haven’t changed the provided `.tfvars`, you can choose from `us-east1-b` , `us-east1-c` or `us-east1-d`

**What tags would you like to be set on the BOSH VM?** `nattable`
This enables the route for egress traffic from the proto BOSH (and all other deployments water-falling from the proto BOSH)

**Would you like to edit the environment file?** `n`
Unless you have some extra Genesis-related configuration changes to add, the wizard has covered everything necessary for our deployment.

And, that’s it! You’ll see a `<environment name>.yml` file in your directory, which Genesis uses for deploying. To deploy your proto BOSH, run `genesis deploy <environment name>.yml`

Once the deploy has finished, you need to upload a stemcell to the director. We recommend version 

### Uploading Cloud Config

Let’s switch gears a bit and open a terminal window to the `codex/terraform/google`. With the proto BOSH deployed, we now want to upload the appropriate cloud config Terraform generated. The file  `genesis-ops.yml` contains just that, and we want to get it over to the bastion host. The easiest way would be a copy&paste-into-new-file operation, but feel free to do that however you’d like. We’ll be copy&pasting:

First, copy the contents of `genesis-ops.yml`  (`cat genesis-ops.yml | pbcopy`  for you Mac folk), and then return to the bastion host. `nano ~/bosh-deployments/cc-ops.yml` and paste the contents inside and save via ctrl-x.

Then, run `bosh ucc cc-ops.yml`  which will upload the cloud config to the proto BOSH. 


### Deploying Vault
Now that your proto BOSH is stood up,  we want to migrate the temporary local Vault we have into a permanent install.

Go to the home directory of the bastion host, and run `genesis init -k vault` to setup a deployment folder for Vault, and cd into it: `cd vault-deployments`

From the `vault-deployments` folder, run `genesis new <environment name>`. This environment name should match the one you previously chose for the proto BOSH. You’ll now run through the Genesis wizard for a Vault deployment, here’s what you should input:

**Which vault would you like to target?** `dev`
This is the local Vault from earlier.

And that’s it! There’s now a `<environment name>.yml` file within your deployments folder. To deploy Vault, run `genesis deploy <environment name>.yml`

Once that’s fully deployed,  we need to migrate the data in the temporary Vault over to the new one we’ve stood up. To do so, run `safe -T dev export | safe -T <environment name> import`. This will pipe all the secrets from the local Vault to the permanent one.




ENABLE POSTGRES EXTENSIONS
-> pat msg and get db: &db database: (without ccdb)
UPDATE: db, ccdb, uaadb, diegodb
_var_vcap_jobs_cloud_controller_ng_config_
sudo apt-get install postgres-client
`create extension citext;`
from 

TERRAFORM CHANGED TO BIGGER POSTGRES TO ACCODMATE CONNECTION COUNT

## Definitions & “What...?”
**GCP** Google Cloud Platform. It’s Google’s cloud services offering, a la Amazon’s AWS. 

**Terraform** Tool used to setup the infrastructure surrounding all your cloud VMs. You _could_ spend all week working with Google Console web UI to setup the network tiles & databases, or you could write a Terraform ruleset to do it for you in a consistent, predictable and automated way. Plus you can use it to generate cloud-configs easily. 



