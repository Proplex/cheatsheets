# CF Genesis Kit Explanation
#work #genesis #cf

A per-VM drill-down of each one’s responsibilities. Something wrong, outdated, or needs clarification? Message me in Slack :) 

## Access VM
Has two jobs: `file_server` and `ssh_proxy`, which serves all static files used by CF apps and brokers SSH connections for `cf ssh` respectively. Both these jobs come from  `diego-release` 

- - - -

## API VM
* `cloud_controller_ng` (from `capi-release`)
The actual Cloud Controller API which provides all the REST endpoints for managing CF. If you’re using the CLI, you’re hitting all the endpoints that `cloud_controller_ng` exposes. It’s where all the operator orchestration occurs.

It maintains all data about spaces|organizations|etc. in a database provided by the Genesis deployment manifest.

* `cloud_controller_worker` (from `capi-release`)
Responsible for executing the actual tasks that are called by the API. Abstraction layer between the RESTful endpoints and the actual work that gets done.

* `cloud_controller_clock` (from `capi-release`)
A scheduler that triggers periodic tasks for the Cloud Controller.

* `policy-server` (from `cf-networking-release`)
Responsible for maintaining all policies about internal networking and providing that to the `vxlan-policy-server` located on the `cell` VM.

* `policy-server-internal` (from `cf-networking-release`)
Funky abstraction layer that `policy-server` uses to communicate with `vxlan-policy-agent`

- - - -

## BBS VM
* `bbs`  (from `diego-release`)
Maintains Diego state within the database chosen during Genesis deploy (which could be external MySQL, PSQL, or internal PSQL.

* `silk-controller` (from `cf-networking-release`)
Maintains all Silk policies and state within the database chosen during Genesis deploy.

* `locket` (from `diego-release`)
A distributed locking service to prevent resource usage collision. Maintains state within the database chosen during Genesis deploy.

- - - -

## Blobstore VM
`blobstore` (from `capi-release`)
This VM is optional, and only deployed if the Genesis manifest requests a `local`  blobstore. This VM stores all static app files. Typically, you use the Blobstore offering provided by the IaaS, but if you’re using vSphere to deploy CF you’ll need to roll your own blobstore. Hence, this VM.

- - - -

## Cell VM
Cell is where CF apps are containerized & ran.  It has the following CF jobs:

* `garden` (from `garden-runc-release`)
The actual container runtime within Diego.

* `rep` (from `diego-release`)
The container manager, responsible for placing containers into which cell; taking app stdout/stderr and handing it to syslog collection & app log collection.

* `cflinuxfs2-rootfs-setup` (from `cflinuxfs2-release`)
The filesystem where apps are mounted in. The filesystem also contains a whole linux operating system. In our case, it’s Ubuntu Trusty 14.04.

* `garden-cni` (from `cf-networking-release`)
The Garden Container Networking Interface. Responsible for container (app) networking. It’s running Silk which is Cloud Foundry’s own policy-based network interface. 

* `netmon` (from `cf-networking-release`)
A monitoring tool, used for traffic capture and protocol analysis.

* `vxlan-policy-agent` (from `cf-networking-release`)
Responsible for enforcing network policy rules  between applications. Discovers network policies from `policy-server`, sets `iptables` rules within the Diego cell for ingress traffic, and tags egress traffic with UUIDs.

* `silk-daemon` (from `cf-networking-release`)
Responsible for providing/assigning IP addresses to the app containers.

* `cni` (from `cf-networking-release`)
Abstraction layer for Silk to communicate to Diego via the CNI API.

- - - -

## Consul etcd VM
* `etcd` (from `etcd-release`)
A distributed key-value store. Fully superseded by Consul, but still in the kit while we find time to remove it. Still used by some other VMs in our current Genesis CF kit.

- - - -

## Diego VM
* `auctioneer` (from `diego-release`)
Responsible for auctioning off new app containers, each auction is taken by a Rep process running on a `cell` VM.

* `tps` (from `capi-release`)
Provides information about apps (instance count, etc.) to the CF API for commands like `cf app`

* `cc_uploader` (from `capi-release`)
Abstraction layer for `rep` (essentially, `diego`), to upload files to the Cloud Controller’s blobstore.

* `route_emitter` (from `diego-release`)
Monitors app states, and broadcasts route registration and unregistration messages for app instances.

* `scheduler` (from `cf-syslog-drain`)
Sits on app instances and periodically (on a schedule that’s algorithmically [automatically] determined) sends all syslog entries to `syslogger`

- - - -

## Doppler VM
* `doppler` (from `loggregator-release`)
Intermediary between the `metron-agent`  (which grabs the stdout/stdin of the app instances) and `loggregator`.

- - - -

## HAProxy VM
* `haproxy`  (from `haproxy-release`)
Acts as the frontend load balancer for the CF API and all external routes. Can be optionally deployed. The actual, public carts are placed on these VMs for external traffic. 

- - - -

## Loggregator Trafficcontroller VM

* `loggregator_trafficcontroller` (from `loggregator-release`)
Handles client requests for logs (`cf logs <app>`) by ferrying content from the `doppler` processes. Also responsible for the firehose.

* `reverse_log_proxy` (from `loggregator-release`)
Collects logs from the `doppler` processes and forwards them to the scalable syslog adapter.

- - - -

## NATS VM
* `nats` (from `nats-release`)
A distributed pubsub messaging queue, used for any CF component to communicate across the internal CF network. Route registration, for example, uses NATS for communicating route setups.

- - - -

## UAA VM
* `uaa` (from `uaa-release`)
The identity management service for CF, handles tenancy, privileges, and authentication. It’s a sprawling release, needless to say, and any questions about it are best directed to their [Github Repo](https://github.com/cloudfoundry/uaa)

- - - -


## Tasks Found in Various VMs
* `consul-agent`
Used for service discovery, distributed lock system, and key-value store. Allows passing of information between VMs easily. 

* `route_registrar` (from `routing-release`)
Responsible for periodically broadcasting route information to the `go-router`. Apps and services use this for external domain access (i.e. `my-app.apps.cfdomain.com`)

* `statsd_injector` (from `statsd_injector`)
Responsible for grabbing metrics from a VM and sending them to loggregator for use in monitoring software.
