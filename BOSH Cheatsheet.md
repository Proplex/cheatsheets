# BOSH Cheatsheet
#work  #cheatsheet #bosh

## Targeting BOSH Directors & Deployments
The default way to target things would be to `export BOSH_DIRECTOR=<ip or name>` and `exoprt BOSH_DEPLOYMENT=<name>`, but a much more fluid method would be to use [James’ BOSH Wrapper](https://raw.githubusercontent.com/jhunt/env/master/bash/bosh). Throw that somehwhere in `~/` and source it via `.bash_rc`. 

Then, to start targeting, simply run `bosh is <director> <deployment>`

## Running Errands
Things like smoke-tests (within CF) are run as an errand via BOSH. To run one, first target the BOSH director & deployment and run:
 `bosh run-errand <name>`