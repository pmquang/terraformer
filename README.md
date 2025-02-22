# Terraformer

[![Build Status](https://travis-ci.com/GoogleCloudPlatform/terraformer.svg?branch=master)](https://travis-ci.com/GoogleCloudPlatform/terraformer)
[![Go Report Card](https://goreportcard.com/badge/github.com/GoogleCloudPlatform/terraformer)](https://goreportcard.com/report/github.com/GoogleCloudPlatform/terraformer)

A CLI tool that generates `tf` and `tfstate` files based on existing infrastructure
(reverse Terraform).

*   Disclaimer: This is not an official Google product
*   Status: beta - we still need to improve documentation, squash some bugs, etc...
*   Created by: Waze SRE

![Waze SRE logo](docs/waze-sre-logo.png)


# Table of Contents
- [Capabilities](#capabilities)
- [Installation](#installation)
- [Supported providers](/providers)
    * [Google Cloud](#use-with-gcp)
    * [AWS](#use-with-aws)
    * [OpenStack](#use-with-openstack)
    * [Kubernetes](#use-with-kubernetes)
    * [Github](#use-with-github)
    * [Datadog](#use-with-datadog)
    * [Cloudflare](#use-with-cloudflare)
    * [Logzio](#use-with-logzio)
    * [NewRelic](#use-with-newrelic)
- [Contributing](#contributing)
- [Developing](#developing)
- [Infrastructure](#infrastructure)

## Capabilities

1.  Generate `tf` + `tfstate` files from existing infrastructure for all
    supported objects by resource.
2.  Remote state can be uploaded to a GCS bucket.
3.  Connect between resources with `terraform_remote_state` (local and bucket).
4.  Save `tf` files using a custom folder tree pattern.
5.  Import by resource name and type.

Terraformer uses terraform providers and is designed to easily support newly added resources.
To upgrade resources with new fields, all you need to do is upgrade the relevant terraform providers.
```
Import current State to terraform configuration from google cloud

Usage:
   import google [flags]
   import google [command]

Available Commands:
  list        List supported resources for google provider

Flags:
  -b, --bucket string         gs://terraform-state
  -c, --connect                (default true)
  -f, --filter strings        google_compute_firewall=id1:id2:id4
  -h, --help                  help for google
  -o, --path-output string     (default "generated")
  -p, --path-pattern string   {output}/{provider}/custom/{service}/ (default "{output}/{provider}/{service}/")
      --projects strings
  -z, --regions strings       europe-west1, (default [global])
  -r, --resources strings     firewalls,networks
  -s, --state string          local or bucket (default "local")

Use " import google [command] --help" for more information about a command.
```
#### Permissions

Read-only permissions

#### Filtering

Filters are a way to choose which resources `terraformer` imports.

For example:
```
terraformer import aws --resources=vpc,subnet --filter=aws_vpc=myvpcid --regions=eu-west-1
```
will only import the vpc with id `myvpcid`.

##### Resources ID

Filtering is based on Terraform resource ID patterns. To find valid ID patterns for your resource, check the import part of [Terraform documentation][terraform-providers].

[terraform-providers]: https://www.terraform.io/docs/providers/

#### Planning

The `plan` command generates a planfile that contains all the resources set to be imported. By modifying the planfile before running the `import` command, you can rename or filter the resources you'd like to import.

The rest of subcommands and parameters are identical to the `import` command.

```
$ terraformer plan google --resources=networks,firewalls --projects=my-project --zone=europe-west1-d
(snip)

Saving planfile to generated/google/my-project/terraformer/plan.json
```

After reviewing/customizing the planfile, begin the import by running `import plan`.

```
$ terraformer import plan generated/google/my-project/terraformer/plan.json
```

### Installation
From source:
1.  Run `git clone <terraformer repo>`
2.  Run `GO111MODULE=on go mod vendor`
3.  Run `go build -v`
4.  Run ```terraform init``` against an ```init.tf``` file to install the plugins required for your platform. For example, if you need plugins for the google provider, ```init.tf``` should contain:
```
provider "google" {}
```
Or alternatively

4.  Copy your Terraform provider's plugin(s) to folder
    `~/.terraform.d/plugins/{darwin,linux}_amd64/`, as appropriate.

From Releases:

* Linux
```
curl -LO https://github.com/GoogleCloudPlatform/terraformer/releases/download/$(curl -s https://api.github.com/repos/GoogleCloudPlatform/terraformer/releases/latest | grep tag_name | cut -d '"' -f 4)/terraformer-linux-amd64
chmod +x terraformer-linux-amd64
sudo mv terraformer-linux-amd64 /usr/local/bin/terraformer
```
* MacOS
```
curl -LO https://github.com/GoogleCloudPlatform/terraformer/releases/download/$(curl -s https://api.github.com/repos/GoogleCloudPlatform/terraformer/releases/latest | grep tag_name | cut -d '"' -f 4)/terraformer-darwin-amd64
chmod +x terraformer-darwin-amd64
sudo mv terraformer-darwin-amd64 /usr/local/bin/terraformer
```

#### Using a package manager

If you want to use a package manager:

- [Homebrew](https://brew.sh/) users can use `brew install terraformer`.

Links to download terraform providers:
* google cloud provider >2.0.0 - [here](https://releases.hashicorp.com/terraform-provider-google/)
* aws provider >1.56.0 - [here](https://releases.hashicorp.com/terraform-provider-aws/)
* openstack provider >1.17.0 - [here](https://releases.hashicorp.com/terraform-provider-openstack/)
* kubernetes provider >=1.4.0 - [here](https://releases.hashicorp.com/terraform-provider-kubernetes/)
* github provider >=2.0.0 - [here](https://releases.hashicorp.com/terraform-provider-github/)
* datadog provider >1.19.0 - [here](https://releases.hashicorp.com/terraform-provider-datadog/)
* logzio provider >=1.1.1 - [here](https://github.com/jonboydell/logzio_terraform_provider/)

Information on provider plugins:
https://www.terraform.io/docs/configuration/providers.html

### Use with GCP
[![asciicast](https://asciinema.org/a/243961.svg)](https://asciinema.org/a/243961)
Example:

```
terraformer import google --resources=gcs,forwardingRules,httpHealthChecks --connect=true --regions=europe-west1,europe-west4 --projects=aaa,fff
terraformer import google --resources=gcs,forwardingRules,httpHealthChecks --filter=google_compute_firewall=rule1:rule2:rule3 --regions=europe-west1 --projects=aaa,fff
```

List of supported GCP services:

*   `addresses`
    * `google_compute_address`
*   `autoscalers`
    * `google_compute_autoscaler`
*   `backendBuckets`
    * `google_compute_backend_bucket`
*   `backendServices`
    * `google_compute_backend_service`
*   `bigQuery`
    * `google_bigquery_dataset`
    * `google_bigquery_table`
*   `schedulerJobs`
    * `google_cloud_scheduler_job`
*   `disks`
    * `google_compute_disk`
*   `firewalls`
    * `google_compute_firewall`
*   `forwardingRules`
    * `google_compute_forwarding_rule`
*   `globalAddresses`
    * `google_compute_global_address`
*   `globalForwardingRules`
    * `google_compute_global_forwarding_rule`
*   `healthChecks`
    * `google_compute_health_check`
*   `httpHealthChecks`
    * `google_compute_http_health_check`
*   `httpsHealthChecks`
    * `google_compute_https_health_check`
*   `images`
    * `google_compute_image`
*   `instanceGroupManagers`
    * `google_compute_instance_group_manager`
*   `instanceGroups`
    * `google_compute_instance_group`
*   `instanceTemplates`
    * `google_compute_instance_template`
*   `instances`
    * `google_compute_instance`
*   `interconnectAttachments`
    * `google_compute_interconnect_attachment`
*   `memoryStore`
    * `google_redis_instance`
*   `networks`
    * `google_compute_network`
*   `nodeGroups`
    * `google_compute_node_group`
*   `nodeTemplates`
    * `google_compute_node_template`
*   `regionAutoscalers`
    * `google_compute_region_autoscaler`
*   `regionBackendServices`
    * `google_compute_region_backend_service`
*   `regionDisks`
    * `google_compute_region_disk`
*   `regionInstanceGroupManagers`
    * `google_compute_region_instance_group_manager`
*   `routers`
    * `google_compute_router`
*   `routes`
    * `google_compute_route`
*   `securityPolicies`
    * `google_compute_security_policy`
*   `sslPolicies`
    * `google_compute_ssl_policy`
*   `subnetworks`
    * `google_compute_subnetwork`
*   `targetHttpProxies`
    * `google_compute_target_http_proxy`
*   `targetHttpsProxies`
    * `google_compute_target_https_proxy`
*   `targetInstances`
    * `google_compute_target_instance`
*   `targetPools`
    * `google_compute_target_pool`
*   `targetSslProxies`
    * `google_compute_target_ssl_proxy`
*   `targetTcpProxies`
    * `google_compute_target_tcp_proxy`
*   `targetVpnGateways`
    * `google_compute_vpn_gateway`
*   `urlMaps`
    * `google_compute_url_map`
*   `vpnTunnels`
    * `google_compute_vpn_tunnel`
*   `gke`
    * `google_container_cluster`
    * `google_container_node_pool`
*   `pubsub`
    * `google_pubsub_subscription`
    * `google_pubsub_topic`
*   `dataProc`
    * `google_dataproc_cluster`
*   `cloudFunctions`
    * `google_cloudfunctions_function`
*   `gcs`
    * `google_storage_bucket`
    * `google_storage_bucket_acl`
    * `google_storage_default_object_acl`
    * `google_storage_bucket_iam_binding`
    * `google_storage_bucket_iam_member`
    * `google_storage_bucket_iam_policy`
    * `google_storage_notification`
*   `monitoring`
    * `google_monitoring_alert_policy`
    * `google_monitoring_group`
    * `google_monitoring_notification_channel`
    * `google_monitoring_uptime_check_config`
*   `dns`
    * `google_dns_managed_zone`
    * `google_dns_record_set`
*   `cloudsql`
    * `google_sql_database_instance`
    * `google_sql_database`
*   `kms`
    * `google_kms_key_ring`
    * `google_kms_crypto_key`
*   `project`
    * `google_project`
*   `logging`
    * `google_logging_metric`

Your `tf` and `tfstate` files are written by default to
`generated/gcp/zone/service`.

### Use with AWS

Example:

```
 terraformer import aws --resources=vpc,subnet --connect=true --regions=eu-west-1 --profile=prod
 terraformer import aws --resources=vpc,subnet --filter=aws_vpc=vpc_id1:vpc_id2:vpc_id3 --regions=eu-west-1
```

List of supported AWS services:

*   `elb`
    * `aws_elb`
*   `alb`
    * `aws_lb`
    * `aws_lb_listener`
    * `aws_lb_listener_rule`
    * `aws_lb_listener_certificate`
    * `aws_lb_target_group`
    * `aws_lb_target_group_attachment`
*   `auto_scaling`
    * `aws_autoscaling_group`
    * `aws_launch_configuration`
    * `aws_launch_template`
*   `rds`
    * `aws_db_instance`
    * `aws_db_parameter_group`
    * `aws_db_subnet_group`
    * `aws_db_option_group`
    * `aws_db_event_subscription`
*   `iam`
    * `aws_iam_role`
    * `aws_iam_role_policy`
    * `aws_iam_user`
    * `aws_iam_user_group_membership`
    * `aws_iam_user_policy`
    * `aws_iam_policy_attachment`
    * `aws_iam_policy`
    * `aws_iam_group`
    * `aws_iam_group_membership`
    * `aws_iam_group_policy`
*   `igw`
    * `aws_internet_gateway`
*   `nacl`
    * `aws_network_acl`
*   `s3`
    * `aws_s3_bucket`
    * `aws_s3_bucket_policy`
*   `sg`
    * `aws_security_group`
*   `subnet`
    * `aws_subnet`
*   `vpc`
    * `aws_vpc`
*   `vpn_connection`
    * `aws_vpn_connection`
*   `vpn_gateway`
    * `aws_vpn_gateway`
*   `route53`
    * `aws_route53_zone`
    * `aws_route53_record`
*   `acm`
    * `aws_acm_certificate`
*   `elasticache`
    * `aws_elasticache_cluster`
    * `aws_elasticache_parameter_group`
    * `aws_elasticache_subnet_group`
    * `aws_elasticache_replication_group`
*   `cloudfront`
    * `aws_cloudfront_distribution`
*   `ec2_instance`
    * `aws_instance`
*   `firehose`
    * `aws_kinesis_firehose_delivery_stream`
*   `glue`
    * `glue_crawler`
*   `route_table`
    * `aws_route_table`

### Use with OpenStack

Example:

```
 terraformer import openstack --resources=compute,networking --regions=RegionOne
```

List of supported OpenStack services:

*   `compute`
    * `openstack_compute_instance_v2`
*   `networking`
    * `openstack_networking_secgroup_v2`
    * `openstack_networking_secgroup_rule_v2`
*   `blockstorage`
    * `openstack_blockstorage_volume_v1`
    * `openstack_blockstorage_volume_v2`
    * `openstack_blockstorage_volume_v3`

### Use with Kubernetes

Example:

```
 terraformer import kubernetes --resources=deployments,services,storageclasses
 terraformer import kubernetes --resources=deployments,services,storageclasses --filter=kubernetes_deployment=name1:name2:name3
```

All kubernetes resources that are currently supported by the kubernetes provider, are also supported by this module. Here is the list of resources which are currently supported by kubernetes provider v.1.4:

* `clusterrolebinding`
  * `kubernetes_cluster_role_binding`
* `configmaps`
  * `kubernetes_config_map`
* `deployments`
  * `kubernetes_deployment`
* `horizontalpodautoscalers`
  * `kubernetes_horizontal_pod_autoscaler`
* `limitranges`
  * `kubernetes_limit_range`
* `namespaces`
  * `kubernetes_namespace`
* `persistentvolumes`
  * `kubernetes_persistent_volume`
* `persistentvolumeclaims`
  * `kubernetes_persistent_volume_claim`
* `pods`
  * `kubernetes_pod`
* `replicationcontrollers`
  * `kubernetes_replication_controller`
* `resourcequotas`
  * `kubernetes_resource_quota`
* `secrets`
  * `kubernetes_secret`
* `services`
  * `kubernetes_service`
* `serviceaccounts`
  * `kubernetes_service_account`
* `statefulsets`
  * `kubernetes_stateful_set`
* `storageclasses`
  * `kubernetes_storage_class`

#### Known issues

* Terraform kubernetes provider is rejecting resources with ":" characters in their names (as they don't meet DNS-1123), while it's allowed for certain types in kubernetes, e.g. ClusterRoleBinding.
* Because terraform flatmap uses "." to detect the keys for unflattening the maps, some keys with "." in their names are being considered as the maps.
* Since the library assumes empty strings to be empty values (not "0"), there are some issues with optional integer keys that are restricted to be positive.

### Use with Github

Example:

```
 ./terraformer import github --organizations=YOUR_ORGANIZATION --resources=repositories --token=YOUR_TOKEN // or GITHUB_TOKEN in env
 ./terraformer import github --organizations=YOUR_ORGANIZATION --resources=repositories --filter=github_repository=id1:id2:id4 --token=YOUR_TOKEN // or GITHUB_TOKEN in env
```

Supports only organizational resources. List of supported resources:

* `repositories`
    * `github_repository`
    * `github_repository_webhook`
    * `github_branch_protection`
    * `github_repository_collaborator`
    * `github_repository_deploy_key`
* `teams`
    * `github_team`
    * `github_team_membership`
    * `github_team_repository`
* `members`
    * `github_membership`
* `organization_webhooks`
    * `github_organization_webhook`

Notes:
* Terraformer can't get webhook secrets from the github API. If you use a secret token in any of your webhooks, running `terraform plan` will result in a change being detected:
=> `configuration.#: "1" => "0"` in tfstate only.

### Use with Datadog

Example:

```
 ./terraformer import datadog --resources=monitor --api-key=YOUR_DATADOG_API_KEY // or DATADOG_API_KEY in env --app-key=YOUR_DATADOG_APP_KEY // or DATADOG_APP_KEY in env
 ./terraformer import datadog --resources=monitor --filter=datadog_monitor=id1:id2:id4 --api-key=YOUR_DATADOG_API_KEY // or DATADOG_API_KEY in env --app-key=YOUR_DATADOG_APP_KEY // or DATADOG_APP_KEY in env
```

List of supported Datadog services:

* `downtime`
    * `datadog_downtime`
* `monitor`
    * `datadog_monitor`
* `screenboard`
    * `datadog_screenboard`
* `synthetics`
    * `datadog_synthetics_test`
* `timeboard`
    * `datadog_timeboard`
* `user`
    * `datadog_user`


### Use with Cloudflare

Example:
```
CLOUDFLARE_TOKEN=[CLOUDFLARE_API_TOKEN]
CLOUDFLARE_EMAIL=[CLOUDFLARE_EMAIL]
 ./terraformer import cloudflare --resources=firewall,dns
```

List of supported Cloudflare services:

* `firewall`
  * `cloudflare_access_rule`
  * `cloudflare_filter`
  * `cloudflare_firewall_rule`
  * `cloudflare_zone_lockdown`
* `dns`
  * `cloudflare_zone`
  * `cloudflare_record`
* `access`
  * `cloudflare_access_application`


### Use with Logz.io

Example:

```
 LOGZIO_API_TOKEN=foobar LOGZIO_BASE_URL=https://api-eu.logz.io ./terraformer import logzio -r=alerts,alert_notification_endpoints // Import Logz.io alerts and alert notification endpoints
```

List of supported Logz.io resources:

* `alerts`
    * `logzio_alert`
* `alert notification endpoints`
    * `logzio_endpoint`

### Use with NewRelic
Example:

```
NEWRELIC_API_KEY=[API-KEY]
./terraformer import newrelic -r alert,dashboard,infra,synthetics
```

List of supported NewRelic resources:

* `alert`
    * `newrelic_alert_channel`
    * `newrelic_alert_condition`
    * `newrelic_alert_policy`
* `dashboard`
    * `newrelic_dashboard`
* `infra`
    * `newrelic_infra_alert_condition`
* `synthetics`
    * `newrelic_synthetics_monitor`
    * `newrelic_synthetics_alert_condition`

## Contributing

If you have improvements or fixes, we would love to have your contributions.
Please read CONTRIBUTING.md for more information on the process we would like
contributors to follow.

## Developing
Terraformer was built so you can easily add new providers of any kind.

Process for generating `tf` + `tfstate` files:

1.  Call GCP/AWS/other api and get list of resources.
2.  Iterate over resources and take only the ID (we don't need mapping fields!)
3.  Call to provider for readonly fields.
4.  Call to infrastructure and take tf + tfstate.

## Infrastructure

1.  Call to provider using the refresh method and get all data.
2.  Convert refresh data to go struct.
3.  Generate HCL file - `tf` files.
4.  Generate `tfstate` files.

All mapping of resource is made by providers and Terraform. Upgrades are needed only
for providers.

##### GCP compute resources

For GCP compute resources, use generated code from
`providers/gcp/gcp_compute_code_generator`.

To regenerate code:

```
go run providers/gcp/gcp_compute_code_generator/*.go
```

### Similar projects


#### [terraforming](https://github.com/dtan4/terraforming)

##### Terraformer Benefits

* Simpler to add new providers and resources - already supports AWS, GCP, Github, Kubernetes, and Openstack. Terraforming supports only AWS.
* Better support for HCL + tfstate, including updates for Terraform 0.12
* If a provider adds new attributes to a resource, there is no need change Terraformer code - just update the terraform provider on your laptop.
* Automatically supports connections between resources in HCL files

##### Comparison

Terraforming gets all attributes from cloud APIs and creates HCL and tfstate files with templating. Each attribute in the API needs to map to attribute in terraform. Generated files from templating can be broken with illegal syntax. When a provider adds new attributes the terraforming code needs to be updated.

Terraformer instead uses terraform provider files for mapping attributes, HCL library from hashicorp, and terraform code.

Look for S3 support in Terraforming here and official s3 support
Terraforming lacks full coverage for resources - as an example you can see that 70% of s3 options are not supported:

* terraforming - https://github.com/dtan4/terraforming/blob/master/lib/terraforming/template/tf/s3.erb
* official s3 support - https://www.terraform.io/docs/providers/aws/r/s3_bucket.html
