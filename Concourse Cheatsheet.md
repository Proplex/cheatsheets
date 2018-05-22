# Concourse Cheatsheet
#concourse #work #cheatsheet

## Targeting a Concourse
This both adds the Concourse URL to `fly targets`  and logins you in.

```
fly -t <target name> -c <URL> login
```

## Getting All Pipelines
Get an overview of all current pipelines and their status

```
fly -t <target name> pipelines
```

## On Redeploy
Make sure to `bosh recreate`  because the workers are bad and that’s really the only fix.
