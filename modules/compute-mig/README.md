# GCE Managed Instance Group module

This module allows creating a managed instance group supporting one or more application versions via instance templates. A health check and an autoscaler can also be optionally created.

This module can be coupled with the [`compute-vm`](../compute-vm) module which can manage instance templates, and the [`net-ilb`](../net-ilb) module to assign the MIG to a backend wired to an Internal Load Balancer. The first use case is shown in the examples below.

## TODO

- [ ] add examples

## Examples

### Simple MIG

```hcl
module "cos-nginx" {
  source = "./modules/cloud-config-container/nginx"
}

module "nginx-template" {
  source     = "./modules/compute-vm"
  project_id = "my-project"
  region     = "europe-west1"
  zone       = "europe-west1-b"
  name       = "ilb-test"
  network_interfaces = [{
    network    = local.network_self_link,
    subnetwork = local.subnetwork_self_link,
    nat        = false,
    addresses  = null
  }]
  boot_disk = {
    image = "projects/cos-cloud/global/images/family/cos-stable"
    type  = "pd-ssd"
    size  = 10
  }
  tags                   = ["http-server", "ssh"]
  use_instance_template  = true
  metadata = {
    user-data = module.cos-nginx.cloud_config
  }
}

module "mig" {
  source = "./modules/compute-mig"
  project_id = "my-project"
  region     = "europe-west1"
  name       = "mig-test"
  target_size   = 2
  versions = {
    default = {
      instance_template = module.nginx-template.template.self_link
      target_type = null
      target_size = null
    }
  }
}
```

<!-- BEGIN TFDOC -->
## Variables

| name | description | type | required | default |
|---|---|:---: |:---:|:---:|
| name | Managed group name. | <code title="">string</code> | ✓ |  |
| project_id | Project id. | <code title="">string</code> | ✓ |  |
| zone | Compute zone. | <code title="">string</code> | ✓ |  |
| *auto_healing_policies* | Auto-healing policies for this group. | <code title="object&#40;&#123;&#10;health_check      &#61; string&#10;initial_delay_sec &#61; number&#10;&#125;&#41;">object({...})</code> |  | <code title="">null</code> |
| *autoscaler_config* | Optional autoscaler configuration. Only one of 'cpu_utilization_target' 'load_balancing_utilization_target' or 'metric' can be not null. | <code title="object&#40;&#123;&#10;max_replicas                      &#61; number&#10;min_replicas                      &#61; number&#10;cooldown_period                   &#61; number&#10;cpu_utilization_target            &#61; number&#10;load_balancing_utilization_target &#61; number&#10;metric &#61; object&#40;&#123;&#10;name                       &#61; string&#10;single_instance_assignment &#61; number&#10;target                     &#61; number&#10;type &#61; string &#35; GAUGE, DELTA_PER_SECOND, DELTA_PER_MINUTE&#10;filter                     &#61; string&#10;&#125;&#41;&#10;&#125;&#41;">object({...})</code> |  | <code title="">null</code> |
| *health_check_config* | Optional auto-created helth check configuration, use the output self-link to set it in the auto healing policy. Refer to examples for usage. | <code title="object&#40;&#123;&#10;type &#61; string      &#35; http https tcp ssl http2&#10;check   &#61; map&#40;any&#41;    &#35; actual health check block attributes&#10;config  &#61; map&#40;number&#41; &#35; interval, thresholds, timeout&#10;logging &#61; bool&#10;&#125;&#41;">object({...})</code> |  | <code title="">null</code> |
| *named_ports* | Named ports. | <code title="map&#40;number&#41;">map(number)</code> |  | <code title="">null</code> |
| *region* | Compute region, set to use regional group. | <code title="">string</code> |  | <code title="">null</code> |
| *target_pools* | Optional list of URLs for target pools to which new instances in the group are added. | <code title="list&#40;string&#41;">list(string)</code> |  | <code title="">null</code> |
| *target_size* | Group target size, leave null when using an autoscaler. | <code title="">number</code> |  | <code title="">null</code> |
| *update_policy* | Update policy. Type can be 'OPPORTUNISTIC' or 'PROACTIVE', action 'REPLACE' or 'restart', surge type 'fixed' or 'percent'. | <code title="object&#40;&#123;&#10;type &#61; string &#35; OPPORTUNISTIC &#124; PROACTIVE&#10;minimal_action       &#61; string &#35; REPLACE &#124; RESTART&#10;min_ready_sec        &#61; number&#10;max_surge_type       &#61; string &#35; fixed &#124; percent&#10;max_surge            &#61; number&#10;max_unavailable_type &#61; string&#10;max_unavailable      &#61; number&#10;&#125;&#41;">object({...})</code> |  | <code title="">null</code> |
| *versions* | Application versions, target_type is either 'fixed' or 'percent'. | <code title="map&#40;object&#40;&#123;&#10;instance_template &#61; string&#10;target_type       &#61; string &#35; fixed &#124; percent&#10;target_size       &#61; number&#10;&#125;&#41;&#41;">map(object({...}))</code> |  | <code title="">null</code> |
| *wait_for_instances* | Wait for all instances to be created/updated before returning. | <code title="">bool</code> |  | <code title="">null</code> |

## Outputs

| name | description | sensitive |
|---|---|:---:|
| autoscaler | Auto-created autoscaler resource. |  |
| group_manager | Instance group resource. |  |
| health_check | Auto-created health-check resource. |  |
<!-- END TFDOC -->

## TODO

- [ ] add support for instance groups