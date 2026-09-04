# Deploy Black/White Lists

## Application Scenario

Advanced Anti-DDoS (AAD) is a professional DDoS protection service provided by Huawei Cloud, designed to protect Internet servers and applications from distributed denial-of-service (DDoS) attacks and other malicious traffic. AAD provides comprehensive protection capabilities, including DDoS traffic cleaning, CC (Challenge Collapsar) attack protection, and intelligent traffic analysis, ensuring the availability and stability of online services.

This best practice will introduce how to use Terraform to create an AAD instance and configure black/white lists, helping you quickly block malicious IP addresses and allow trusted IP addresses to enhance the security protection capability of your business.

## Related Resources/Data Sources

This best practice involves the following main resources:

### Resources

- [Advanced Anti-DDoS Instance (huaweicloud_aad_instance)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/aad_instance)
- [Advanced Anti-DDoS Black/White List (huaweicloud_aad_black_white_list)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/aad_black_white_list)

### Resource/Data Source Dependencies

```
huaweicloud_aad_instance
    ├── huaweicloud_aad_black_white_list.black
    └── huaweicloud_aad_black_white_list.white
```

## Operation Steps

### 1. Script Preparation

Prepare the TF file (such as main.tf) for writing the current best practice script in the specified working directory, and ensure that it (or other TF files in the same directory) contains the provider version declaration and Huawei Cloud authentication information required for deploying resources.
For configuration introduction, refer to [Preparation Before Deploying Huawei Cloud Resources](../../introductions/prepare_before_deploy.md).

### 2. Create an Advanced Anti-DDoS Instance

Add the following script to the TF file (such as main.tf) to create a new AAD instance when no existing instance ID is specified:

```hcl
# Create an Advanced Anti-DDoS instance in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
variable "instance_config" {
  description = "The configuration for creating a new AAD instance"

  type = object({
    instance_name                  = string
    ip_type                        = number
    resource_region                = string
    instance_access_type           = string
    duration                       = number
    amount                         = number
    period_type                    = number
    service_bandwidth              = number
    basic_bandwidth                = optional(number, null)
    elastic_bandwidth              = optional(number, null)
    basic_qps                      = optional(number, null)
    forwarding_rule                = optional(number, null)
    protected_domain               = optional(number, null)
    elastic_service_bandwidth_type = optional(number, null)
    elastic_service_bandwidth      = optional(number, null)
    protection_package             = optional(string, null)
    enterprise_project_id          = optional(string, null)
  })

  default = null
}

resource "huaweicloud_aad_instance" "test" {
  count = var.instance_id == "" ? 1 : 0

  ip_type                        = var.instance_config.ip_type
  resource_region                = var.instance_config.resource_region
  instance_access_type           = var.instance_config.instance_access_type
  duration                       = var.instance_config.duration
  amount                         = var.instance_config.amount
  instance_name                  = var.instance_config.instance_name
  period_type                    = var.instance_config.period_type
  service_bandwidth              = var.instance_config.service_bandwidth
  basic_bandwidth                = var.instance_config.basic_bandwidth
  elastic_bandwidth              = var.instance_config.elastic_bandwidth
  basic_qps                      = var.instance_config.basic_qps
  forwarding_rule                = var.instance_config.forwarding_rule
  protected_domain               = var.instance_config.protected_domain
  elastic_service_bandwidth_type = var.instance_config.elastic_service_bandwidth_type
  elastic_service_bandwidth      = var.instance_config.elastic_service_bandwidth
  protection_package             = var.instance_config.protection_package
  enterprise_project_id          = var.instance_config.enterprise_project_id

  lifecycle {
    ignore_changes = [
      ip_type,
      resource_region,
      instance_access_type,
      duration,
      amount,
      period_type,
      basic_qps,
      protection_package,
      protected_domain,
      forwarding_rule,
    ]
  }
}
```

**Parameter Description**:
- **count**: Determines whether to create a new instance based on whether the variable instance_id is empty. When instance_id is empty, it creates; otherwise, it does not.
- **ip_type**: Assigned by referencing the ip_type in the input variable instance_config, used to specify the IP type.
- **resource_region**: Assigned by referencing the resource_region in the input variable instance_config, used to specify the resource region.
- **instance_access_type**: Assigned by referencing the instance_access_type in the input variable instance_config, used to specify the access type.
- **duration**: Assigned by referencing the duration in the input variable instance_config, used to specify the subscription duration.
- **amount**: Assigned by referencing the amount in the input variable instance_config, used to specify the number of instances to purchase.
- **instance_name**: Assigned by referencing the instance_name in the input variable instance_config, used to specify the instance name.
- **period_type**: Assigned by referencing the period_type in the input variable instance_config, used to specify the subscription period type.
- **service_bandwidth**: Assigned by referencing the service_bandwidth in the input variable instance_config, used to specify the service bandwidth.
- **basic_bandwidth**: Assigned by referencing the basic_bandwidth in the input variable instance_config, used to specify the basic bandwidth.
- **elastic_bandwidth**: Assigned by referencing the elastic_bandwidth in the input variable instance_config, used to specify the elastic bandwidth.
- **basic_qps**: Assigned by referencing the basic_qps in the input variable instance_config, used to specify the basic QPS.
- **forwarding_rule**: Assigned by referencing the forwarding_rule in the input variable instance_config, used to specify the number of forwarding rules.
- **protected_domain**: Assigned by referencing the protected_domain in the input variable instance_config, used to specify the number of protected domains.
- **elastic_service_bandwidth_type**: Assigned by referencing the elastic_service_bandwidth_type in the input variable instance_config, used to specify the elastic service bandwidth type.
- **elastic_service_bandwidth**: Assigned by referencing the elastic_service_bandwidth in the input variable instance_config, used to specify the elastic service bandwidth increment.
- **protection_package**: Assigned by referencing the protection_package in the input variable instance_config, used to specify the protection package.
- **enterprise_project_id**: Assigned by referencing the enterprise_project_id in the input variable instance_config, used to specify the enterprise project ID.

### 3. Add Blacklist

Add the following script to the TF file (such as main.tf) to add malicious IP addresses to the blacklist:

```hcl
# Add blacklist in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
variable "blacklist_ips" {
  description = "The list of IP addresses to add to the blacklist"
  type        = list(string)
}

resource "huaweicloud_aad_black_white_list" "black" {
  instance_id = var.instance_id != "" ? var.instance_id : try(huaweicloud_aad_instance.test[0].id, "")
  type        = "black"
  ips         = var.blacklist_ips
}
```

**Parameter Description**:
- **instance_id**: Selects the existing instance ID or the ID of the newly created instance based on whether the variable instance_id is empty.
- **type**: Fixed to "black", indicating the blacklist.
- **ips**: Assigned by referencing the input variable blacklist_ips, used to specify the list of IP addresses to add to the blacklist.

### 4. Add Whitelist

Add the following script to the TF file (such as main.tf) to add trusted IP addresses to the whitelist:

```hcl
# Add whitelist in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
variable "whitelist_ips" {
  description = "The list of IP addresses to add to the whitelist"
  type        = list(string)
}

resource "huaweicloud_aad_black_white_list" "white" {
  instance_id = var.instance_id != "" ? var.instance_id : try(huaweicloud_aad_instance.test[0].id, "")
  type        = "white"
  ips         = var.whitelist_ips
}
```

**Parameter Description**:
- **instance_id**: Selects the existing instance ID or the ID of the newly created instance based on whether the variable instance_id is empty.
- **type**: Fixed to "white", indicating the whitelist.
- **ips**: Assigned by referencing the input variable whitelist_ips, used to specify the list of IP addresses to add to the whitelist.

### 5. Preset Input Parameters Required for Resource Deployment (Optional)

In this practice, some resources and data sources use input variables to assign values to configuration content. These input parameters need to be manually entered during subsequent deployment.
Meanwhile, Terraform provides a method to preset these configurations through `tfvars` files, which can avoid repeated input during each execution.

Create a `terraform.tfvars` file in the working directory. The example content is as follows:

```hcl
# Fill in according to the script variables; use placeholders for sensitive information
instance_config = {
  instance_name        = "aad-instance"
  ip_type              = 0
  resource_region      = "north_china"
  instance_access_type = "1"
  duration             = 1
  amount               = 1
  period_type          = 2
  service_bandwidth    = 10
  basic_bandwidth      = 10
  elastic_bandwidth    = 10
  forwarding_rule      = 50
}

blacklist_ips = ["192.170.1.10", "192.170.1.11"]
whitelist_ips = ["192.170.1.200", "192.170.1.201"]
```

**Usage**:

1. Save the above content as a `terraform.tfvars` file in the working directory (this file name allows Terraform to automatically import the content of the `tfvars` file when executing terraform commands; other names need to add `.auto` before tfvars, such as `variables.auto.tfvars`)
2. Modify the parameter values according to actual needs
3. When executing `terraform plan` or `terraform apply`, Terraform will automatically read the variable values from this file

In addition to using the `terraform.tfvars` file, you can also set variable values in the following ways:

1. Command line parameters: `terraform apply -var="instance_name=my-aad-instance"`
2. Environment variables: `export TF_VAR_instance_name=my-aad-instance`
3. Custom named variable files: `terraform apply -var-file="custom.tfvars"`

> Note: If the same variable is set in multiple ways, Terraform will use the variable values in the following priority: command line parameters > variable files > environment variables > default values.

### 6. Initialize and Apply Terraform Configuration

After completing the above script configuration, execute the following steps to create resources:

1. Run `terraform init` to initialize the environment
2. Run `terraform plan` to view the resource creation plan
3. After confirming the resource plan is correct, run `terraform apply` to start creating the Advanced Anti-DDoS instance and black/white lists
4. Run `terraform show` to view the created Advanced Anti-DDoS instance and black/white lists

## Reference Information

- [Huawei Cloud Advanced Anti-DDoS Product Documentation](https://support.huaweicloud.com/aad/index.html)
- [Huawei Cloud Provider Documentation](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs)
- [Best Practice Source Code Reference For AAD Black/White Lists](https://github.com/huaweicloud/terraform-provider-huaweicloud/tree/master/examples/aad/black-white-lists)
