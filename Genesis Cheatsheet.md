# Genesis Cheatsheet
#work #genesis #cheatsheet

## Helpful Links
### GitHub Repositories/Organizations
* [genesis-community](https://github.com/genesis-community)
* [genesis-community/blacksmith-genesis-kit](https://github.com/genesis-community/shield-genesis-kit)
* [genesis-community/bosh-genesis-kit](https://github.com/genesis-community/bosh-genesis-kit)
* [genesis-community/cf-genesis-kit](https://github.com/genesis-community/cf-genesis-kit)
* [genesis-community/jumpbox-genesis-kit](https://github.com/genesis-community/shield-genesis-kit)
* [genesis-community/shield-genesis-kit](https://github.com/genesis-community/shield-genesis-kit)
* [genesis-community/vault-genesis-kit](https://github.com/genesis-community/vault-genesis-kit)
* [starkandwayne/genesis](https://github.com/starkandwayne/genesis)
### PR & Communication
* [ #genesis on slack](https://starkandwayne.slack.com/messages/C0ASZM12R)
* [complaints](https://devnull-as-a-service.com/dev/null)
- - - -
## Installing Genesis
1. Clone Genesis  `git clone git@github.com:starkandwayne/genesis.git`
2. Inside the newly cloned folder, build the latest Genesis binary  `./pack` and then link it however you want   `cp ./genesis-<commit> /usr/local/bin/genesis`
	1. This is necessary because right now Genesis 2.6 is not ‘public’.  This is subject to change in the future.

## Getting Started: Installing BOSH
Setup a temporary Vault with safe:  `safe local --memory --as dev`. This will create a memory-backed Vault to temporarily store BOSH information. Run that command in a separate terminal because it needs to remain a background process. 
 **Make sure you’re using a Vault version <1.0.0**, API changes by Vault broke Safe compatibility. 

In whatever directory you want to get started in, run  `genesis init -k bosh` 

That’ll setup a subfolder named  `bosh-deployments`  which is a Git repo and should be pushed to some Git platform for safe keeping, revision history, etc.

 `cd bosh-deployments` and then run `genesis new <name>`. This will create a new Genesis deployment with the name you’ve specified, and  then run you through an interactive prompt asking for other information. Select `dev` for the Known Vault Targets (that’s the Vault we setup from above), and then answer the questions according to the environment you’ll be working with.

Once the wizard is complete, run `genesis deploy`  to have Genesis to deploy the BOSH according to your configured settings.

Finally, run  `genesis do <name> -- upload-stemcells` , which will ask you what stemcells you want. 

Once BOSH is deployed, you’ll then want to setup a permanent Vault.

## Getting Started: Installing Vault
Once you’ve got BOSH setup, you’ll want to migrate your temporary Vault server to something that won’t lose all of its data on exit.

From the working directory where you normally run `genesis init` (if you’re in `bosh-deployments` right now, then `cd ../`), run `genesis init -k vault`. This will download the Vault Genesis Kit.

Within `vault-deployments`, run `genesis new`  to get started and answer some basic questions. Then,  `genesis deploy`. You may have to edit the `<name>.yml` file to align with your Cloud Config.

Once Vault is deployed & running, you should save the unseal keys & token in 1Password. Then, since the Genesis vault kit automatically targeted & authed you to the new Vault,  run `safe -T dev export | safe -T <name> import` to migrate your Vault data from the memory store into the newly setup Vault. If everything went well, it’s now safe to shut down the local Vault. 

Whenever you’re working in this environment, you’ll want to make sure you’re targeting that Vault that you’ve setup via `safe target <name>`

## Getting Started: More Kits
By now, things should be more or less familiar. If you want to setup a new kit, simply run `genesis init -k <kit name>` , or clone a Genesis kit repository and run `genesis new`

- - - -
## General Overview of Genesis Deployments
Hierarchy-wise, it looks like this:

(BOSH, Vault,  SHIELD, Concourse, Jumpbox) -> (BOSH, CF)
Control Layer/Ops-Only -> Sites

Each deployment is an environment within Genesis, but really we’re also referring to the higher-level scope of the entire “environment” sometimes.
- - - -
## Useful Commands
* `genesis info <env name>` will pretty-print some information about the deployment
* `genesis do <env name> -- list` will show all the plugin commands you can run
* `genesis decompile-kit <kit name>`  or `genesis decompile-kit <kitname>/<version>` will untar the kit release in your current working directory for editing.
* `genesis rotate secrets` regenerates all the certificates a specific kit uses. It’s very useful but also pretty catastrophic so _know what you’re doing beforehand_
- - - -
## Troubleshooting
* Are you on The Last Stemcell™, ubuntu-trusty 3468? 
* Are you on the latest Genesis? Development moves fast right now, when in doubt `git pull && ./pack` within the Genesis repository.


### Pitfalls I’ve Hit While Deploying with Genesis
_These are here in case you hit something that I’ve dealt with in the past. ¯\_(ツ)__¯/

* Deploying to the Buffalo Lab requires only one availability zone. Some kits (like Vault) will fail to deploy because of this. Just add this to the environment `.yml`

```
params:
  env: buff-lab
  availability_zones: [z1]
```

* When deploying another BOSH inside a BOSH (i.e. the lab setup), then the secondary BOSH needed to be made a `bosh_vm_type: large` for everything to work, otherwise the secondary BOSH would’ve failed to compile anything. Update the environment file like so:
```
params:
  env: buff-lab
  bosh_vm_type: large
```

* If you run the Genesis wizard for BOSH and input the incorrect CPI credentials (user, IP, password). You need to edit the appropriate Vault entry, and re-run `genesis <name> deploy` to publish the changes. The first time you’ll notice you have bad creds is when you go to upload stemcells.

* If setting up CF as a lab instance, make sure `skip_ssl_validation` is set to true in the environment file. Otherwise smoke tests will fail and you’ll sit there for another 30 mins and waiting for a redeploy. It should look like this:
```
params:
  env: buff-lab
  skip_ssl_validation: true
```

- - - -

## Random Information That Doesn’t Fit Anywhere Else
`--` is necessary to get Genesis to stop parsing CLI options after `genesis do` commands, and instead pass them on to the addon hook.







