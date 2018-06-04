# BOSH Cheatsheet
#work  #cheatsheet #bosh

## Targeting BOSH Directors & Deployments
The default way to target things would be to:
```
export BOSH_DIRECTOR=<ip or name>
export BOSH_DEPLOYMENT=<name>
```

However, a much more fluid method is using [James’ BOSH Wrapper](https://raw.githubusercontent.com/jhunt/env/master/bash/bosh). Throw that somewhere in `~/` and source it via `.bash_rc`. 

Then, to start targeting, simply run `bosh is <director> <deployment>`

## Running Errands
Things like smoke-tests (within CF) are run as an errand via BOSH. To run one, first target the BOSH director & deployment and run:
 `bosh run-errand <name>`

## Random Information That Doesn’t Fit Anywhere Else
* When changes are made to a BOSH deployment’s networking configuration, it just rebuilds a new VM rather than patching the existing config.