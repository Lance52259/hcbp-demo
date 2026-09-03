# Deploy AAD Black and White Lists

## Application Scenario

Huawei Cloud AAD (Anti-DDoS) service provides DDoS attack protection for public IPs. This practice guides you through using Terraform to create an AAD instance and configure IP black/white lists to block malicious traffic and allow trusted traffic, ensuring stable business operations.

## Related Resources/Data Sources

This best practice involves the following main resources and data sources:

### Data Sources

No data sources are used in this practice.

### Resources

- [AAD Instance (huaweicloud_aad_instance)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/aad_instance)
- [AAD Black/White List (huaweicloud_aad_black_white_list)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/aad_black_white_list)

### Resource/Data Source Dependencies

```
huaweicloud_aad_instance
    └── huaweicloud_aad_black_white_list
```

## Operation Steps

### 1. Script Preparation

Prepare the TF file (such as main.tf) for writing the current best practice script in the specified working directory, ensuring that it (or other TF files in the same directory) contains the provider version declaration and Huawei Cloud authentication information required for deploying resources.
For configuration introduction, refer to the [Preparation Before Deploying Huawei Cloud Resources](../../introductions/prepare_before_deploy.md) article.

### 2. Create an AAD Instance (Optional)

If you do not have an existing AAD instance, you need to create one first. Add the following script to the TF file (such as main.tf):

```hcl
# Create an AAD instance in the specified region (when the region parameter is omitted, it defaults to the region specified in the current provider block)
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
- **ip_type**: IP type, 0 for IPv4, 1 for IPv6.
- **resource_region**: Resource region. Valid values: north_china, east_china, asia_pacific.
- **instance_access_type**: Access type, 0 for website, 1 for IP access.
- **duration**: Subscription duration.
- **amount**: Number of instances to purchase.
- **instance_name**: Instance name.
- **period_type**: Subscription period type, 2 for month, 3 for year.
- **service_bandwidth**: Service bandwidth in Mbps.
- **basic_bandwidth**: Basic bandwidth in Mbps. Required when resource_region is north_china or east_china; must be empty for asia_pacific.
- **elastic_bandwidth**: Elastic bandwidth in Mbps. Required when resource_region is north_china or east_china; must be empty for asia_pacific.
- **basic_qps**: Service QPS. Required when instance_access_type is 0; must be empty when it is 1.
- **forwarding_rule**: Number of forwarding rules. Required when instance_access_type is 1; must be empty when it is 0.
- **protected_domain**: Number of protected domains. Required when instance_access_type is 0; must be empty when it is 1.
- **elastic_service_bandwidth_type**: Elastic service bandwidth type, 2 for daily 95th percentile, 3 for monthly 95th percentile.
- **elastic_service_bandwidth**: Elastic service bandwidth increment.
- **protection_package**: Protection package. Valid values: basic (insurance protection), unlimited (unlimited protection).
- **enterprise_project_id**: Enterprise project ID.

### 3. Configure Black/White Lists

After creating the AAD instance (or using an existing one), you need to add IP black/white lists. Add the following script to the TF file (such as main.tf):

```hcl
# Add IP addresses to the blacklist in the specified region (when the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_aad_black_white_list" "black" {
  instance_id = var.instance_id != "" ? var.instance_id : try(huaweicloud_aad_instance.test[0].id, "")
  type        = "black"
  ips         = var.blacklist_ips
}

# Add IP addresses to the whitelist in the specified region (when the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_aad_black_white_list" "white" {
  instance_id = var.instance_id != "" ? var.instance_id : try(huaweicloud_aad_instance.test[0].id, "")
  type        = "white"
  ips         = var.whitelist_ips
}
```

**Parameter Description**:
- **instance_id**: AAD instance ID. If you have an existing instance, specify it directly; otherwise, it is obtained via `huaweicloud_aad_instance.test[0].id`.
- **type**: List type, black for blacklist, white for whitelist.
- **ips**: List of IP addresses to add.

### 4. Preset Input Parameters for Resource Deployment (Optional)

In this practice, some resources and data sources use input variables to assign configuration content. These input parameters need to be manually entered during subsequent deployment.
Meanwhile, Terraform provides a way to preset these configurations through `tfvars` files, avoiding repeated input during each execution.

Create a `terraform.tfvars` file in the working directory. Example content is as follows:

```hcl
# Fill in according to the script variables; use placeholders for sensitive information
region_name = "cn-north-4"
access_key  = "your-access-key"
secret_key  = "your-secret-key"

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

1. Save the above content as a `terraform.tfvars` file in the working directory (this file name allows Terraform to automatically import the content of the `tfvars` file when executing Terraform commands; other names need to add `.auto` before tfvars, such as `variables.auto.tfvars`).
2. Modify the parameter values according to actual needs.
3. When executing `terraform plan` or `terraform apply`, Terraform will automatically read the variable values from this file.

In addition to using the `terraform.tfvars` file, you can also set variable values in the following ways:

1. Command line parameters: `terraform apply -var="vpc_name=my-vpc"`
2. Environment variables: `export TF_VAR_vpc_name=my-vpc`
3. Custom-named variable files: `terraform apply -var-file="custom.tfvars"`

> Note: If the same variable is set in multiple ways, Terraform will use the variable value in the following priority: command line parameters > variable files > environment variables > default values.

### 5. Initialize and Apply Terraform Configuration

After completing the above script configuration, execute the following steps to create resources:

1. Run `terraform init` to initialize the environment.
2. Run `terraform plan` to view the resource creation plan.
3. After confirming the resource plan is correct, run `terraform apply` to start creating the AAD instance and black/white lists.
4. Run `terraform show` to view the created AAD instance and black/white lists.

## Reference Information

- [Huawei Cloud AAD Product Documentation](https://support.huaweicloud.com/aad/index.html)
- [Huawei Cloud Provider Documentation](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs)
- [AAD Black/White Lists Best Practice Source Code Reference](https://github.com/huaweicloud/terraform-provider-huaweicloud/tree/master/examples/aad/black-white-lists)
