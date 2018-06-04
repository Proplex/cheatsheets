# Safe/Vault Cheatsheet
#safe #vault #cheatsheet

## Helpful Links
### GitHub Repositories/Organizations
* [hashicorp/vault](https://github.com/hashicorp/vault)
* [starkandwayne/safe](https://github.com/starkandwayne/safe)
### PR & Communication
* No Slack Channel ¯\_(ツ)_/¯ 
* [complaints](https://devnull-as-a-service.com/dev/null)

## Get Information from Safe
_From a file…_
`safe get <path> key@filename`

## Running a local Vault server
`safe local --memory --as dev`  spins up a local Vault that safe automatically targets as `dev` . This should be run in a different terminal session since it’s blocking. 

Sometimes, this command will fail to run because Vault didn’t spin up fast enough. To fix it:
	1. Run  `rm ~/.saferc`
	2. Then run `safe local --memory --as dev` again
Now it should be working. It’s a race condition we’re trying to fix.
