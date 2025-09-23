# Deploy Flow Logs

## Application Scenario

Enterprise Router (ER) is a high-performance, highly available enterprise-grade router service provided by Huawei Cloud, supporting enterprise-level network functions such as multi-VPC interconnection, dedicated line access, and VPN connections. ER service provides flexible routing policies and rich network connectivity capabilities, meeting complex enterprise network architecture requirements.

ER flow logs are an important feature of the ER service, used to record and monitor network traffic information on enterprise routers, including detailed information such as source addresses, destination addresses, protocol types, ports, etc. of data packets. Through flow logs, enterprises can analyze network traffic patterns, monitor network performance, conduct security audits, and troubleshoot issues. This best practice will introduce how to use Terraform to automatically deploy flow logs, including VPC creation, ER instance creation, VPC connection, LTS log group creation, and flow log configuration.

## Related Resources/Data Sources

This best practice involves the following main resources and data sources:

### Data Sources

- [ER Availability Zones Query Data Source (data.huaweicloud_er_availability_zones)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/er_availability_zones)

### Resources

- [VPC Resource (huaweicloud_vpc)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc)
- [VPC Subnet Resource (huaweicloud_vpc_subnet)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc_subnet)
- [ER Instance Resource (huaweicloud_er_instance)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/er_instance)
- [ER VPC Connection Resource (huaweicloud_er_vpc_attachment)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/er_vpc_attachment)
- [LTS Log Group Resource (huaweicloud_lts_group)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/lts_group)
- [LTS Log Stream Resource (huaweicloud_lts_stream)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/lts_stream)
- [ER Flow Log Resource (huaweicloud_er_flow_log)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/er_flow_log)

### Resource/Data Source Dependencies

```
data.huaweicloud_er_availability_zones.test
    └── huaweicloud_er_instance.test

huaweicloud_vpc.test
    ├── huaweicloud_vpc_subnet.test
    │   └── huaweicloud_er_vpc_attachment.test
    └── huaweicloud_er_vpc_attachment.test

huaweicloud_er_instance.test
    └── huaweicloud_er_vpc_attachment.test
        └── huaweicloud_er_flow_log.test

huaweicloud_lts_group.test
    └── huaweicloud_lts_stream.test
        └── huaweicloud_er_flow_log.test
```

## Operation Steps

### 1. Script Preparation

Prepare the TF file (e.g., main.tf) in the specified workspace for writing the current best practice script, ensuring that it (or other TF files in the same directory) contains the provider version declaration and Huawei Cloud authentication information required for deploying resources.
Refer to the "Preparation Before Deploying Huawei Cloud Resources" document for configuration introduction.

### 2. Query ER Availability Zone Information Through Data Source

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to perform a data source query, the results of which are used to create ER instances:

```hcl
# Get all ER availability zone information under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block), used to create ER instances
data "huaweicloud_er_availability_zones" "test" {}
```

**Parameter Description**:
- No additional parameters required, the data source will automatically get all ER availability zone information in the current region

### 3. Create VPC

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to create a VPC resource:

```hcl
variable "vpc_name" {
  description = "VPC name"
  type        = string
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "192.168.0.0/16"
}

# Create a VPC resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_vpc" "test" {
  name = var.vpc_name
  cidr = var.vpc_cidr
}
```

**Parameter Description**:
- **name**: VPC name, assigned by referencing the input variable vpc_name
- **cidr**: VPC CIDR block, assigned by referencing the input variable vpc_cidr

### 4. Create VPC Subnet

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to create a VPC subnet resource:

```hcl
variable "subnet_name" {
  description = "VPC subnet name"
  type        = string
}

variable "subnet_cidr" {
  description = "VPC subnet CIDR block"
  type        = string
  default     = ""
}

variable "subnet_gateway_ip" {
  description = "VPC subnet gateway IP"
  type        = string
  default     = ""
}

# Create a VPC subnet resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_vpc_subnet" "test" {
  vpc_id     = huaweicloud_vpc.test.id
  name       = var.subnet_name
  cidr       = var.subnet_cidr == "" ? cidrsubnet(huaweicloud_vpc.test.cidr, 8, 0) : var.subnet_cidr
  gateway_ip = var.subnet_gateway_ip == "" ? cidrhost(cidrsubnet(huaweicloud_vpc.test.cidr, 8, 0), 1) : var.subnet_gateway_ip
}
```

**Parameter Description**:
- **vpc_id**: VPC ID, assigned by referencing the VPC resource (huaweicloud_vpc.test) ID
- **name**: Subnet name, assigned by referencing the input variable subnet_name
- **cidr**: Subnet CIDR block, prioritizes using input variable, calculates using cidrsubnet function if empty
- **gateway_ip**: Gateway IP, prioritizes using input variable, calculates using cidrhost function if empty

### 5. Create ER Instance

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to create an ER instance resource:

```hcl
variable "er_instance_name" {
  description = "ER instance name"
  type        = string
}

variable "er_instance_asn" {
  description = "ER instance ASN number"
  type        = number
}

# Create an ER instance resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_er_instance" "test" {
  availability_zones = slice(data.huaweicloud_er_availability_zones.test.names, 0, 1)
  name               = var.er_instance_name
  asn                = var.er_instance_asn
}
```

**Parameter Description**:
- **availability_zones**: Availability zone list, using the first result from ER availability zone list query data source
- **name**: Instance name, assigned by referencing the input variable er_instance_name
- **asn**: ASN number, assigned by referencing the input variable er_instance_asn

### 6. Create ER VPC Connection

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to create an ER VPC connection resource:

```hcl
variable "er_vpc_attachment_name" {
  description = "ER VPC connection name"
  type        = string
}

variable "er_vpc_attachment_auto_create_vpc_routes" {
  description = "Whether to automatically create VPC routes"
  type        = bool
  default     = true
}

# Create an ER VPC connection resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_er_vpc_attachment" "test" {
  instance_id = huaweicloud_er_instance.test.id
  vpc_id      = huaweicloud_vpc.test.id
  subnet_id   = huaweicloud_vpc_subnet.test.id

  name                   = var.er_vpc_attachment_name
  auto_create_vpc_routes = var.er_vpc_attachment_auto_create_vpc_routes
}
```

**Parameter Description**:
- **instance_id**: ER instance ID, assigned by referencing the ER instance resource (huaweicloud_er_instance.test) ID
- **vpc_id**: VPC ID, assigned by referencing the VPC resource (huaweicloud_vpc.test) ID
- **subnet_id**: Subnet ID, assigned by referencing the VPC subnet resource (huaweicloud_vpc_subnet.test) ID
- **name**: Connection name, assigned by referencing the input variable er_vpc_attachment_name
- **auto_create_vpc_routes**: Auto create VPC routes, assigned by referencing the input variable er_vpc_attachment_auto_create_vpc_routes

### 7. Create LTS Log Group

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to create an LTS log group resource:

```hcl
variable "lts_group_name" {
  description = "LTS log group name"
  type        = string
}

variable "lts_group_ttl_in_days" {
  description = "Log retention days"
  type        = number
  default     = 7
}

# Create an LTS log group resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_lts_group" "test" {
  group_name  = var.lts_group_name
  ttl_in_days = var.lts_group_ttl_in_days
}
```

**Parameter Description**:
- **group_name**: Log group name, assigned by referencing the input variable lts_group_name
- **ttl_in_days**: Log retention days, assigned by referencing the input variable lts_group_ttl_in_days

### 8. Create LTS Log Stream

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to create an LTS log stream resource:

```hcl
variable "lts_stream_name" {
  description = "LTS log stream name"
  type        = string
}

# Create an LTS log stream resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_lts_stream" "test" {
  group_id    = huaweicloud_lts_group.test.id
  stream_name = var.lts_stream_name
}
```

**Parameter Description**:
- **group_id**: Log group ID, assigned by referencing the LTS log group resource (huaweicloud_lts_group.test) ID
- **stream_name**: Log stream name, assigned by referencing the input variable lts_stream_name

### 9. Create ER Flow Log

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to create an ER flow log resource:

```hcl
variable "er_flow_log_name" {
  description = "ER flow log name"
  type        = string
}

variable "er_flow_log_store_type" {
  description = "Flow log storage type"
  type        = string
  default     = "LTS"
}

variable "er_flow_log_resource_type" {
  description = "Resource type for flow log collection"
  type        = string
  default     = "attachment"
}

# Create an ER flow log resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_er_flow_log" "test" {
  name           = var.er_flow_log_name
  instance_id    = huaweicloud_er_instance.test.id
  log_store_type = var.er_flow_log_store_type
  log_group_id   = huaweicloud_lts_group.test.id
  log_stream_id  = huaweicloud_lts_stream.test.id
  resource_type  = var.er_flow_log_resource_type
  resource_id    = huaweicloud_er_vpc_attachment.test.id
}
```

**Parameter Description**:
- **name**: Flow log name, assigned by referencing the input variable er_flow_log_name
- **instance_id**: ER instance ID, assigned by referencing the ER instance resource (huaweicloud_er_instance.test) ID
- **log_store_type**: Log storage type, assigned by referencing the input variable er_flow_log_store_type
- **log_group_id**: Log group ID, assigned by referencing the LTS log group resource (huaweicloud_lts_group.test) ID
- **log_stream_id**: Log stream ID, assigned by referencing the LTS log stream resource (huaweicloud_lts_stream.test) ID
- **resource_type**: Resource type, assigned by referencing the input variable er_flow_log_resource_type
- **resource_id**: Resource ID, assigned by referencing the ER VPC connection resource (huaweicloud_er_vpc_attachment.test) ID

### 10. Preset Input Parameters Required for Resource Deployment (Optional)

In this practice, some resources and data sources use input variables to assign values to configuration content. These input parameters need to be manually entered during subsequent deployments.
At the same time, Terraform provides a method to preset these configurations through `tfvars` files, which can avoid repeated input during each execution.

Create a `terraform.tfvars` file in the working directory with the following example content:

```hcl
# Authentication information
region_name = "cn-north-4"
access_key  = "your-access-key"
secret_key  = "your-secret-key"

# Network configuration
vpc_name    = "tf_test_er_instance_vpc"
vpc_cidr    = "192.168.0.0/16"
subnet_name = "tf_test_er_instance_subnet"

# ER instance configuration
er_instance_name = "tf_test_er_instance"
er_instance_asn  = 64512

# ER VPC connection configuration
er_vpc_attachment_name = "tf_test_er_vpc_attachment"

# LTS log configuration
lts_group_name  = "tf_test_lts_group"
lts_stream_name = "tf_test_lts_stream"

# ER flow log configuration
er_flow_log_name = "tf_test_er_flow_log"
```

**Usage**:

1. Save the above content as `terraform.tfvars` file in the working directory (this file name allows users to automatically import the content of this `tfvars` file when executing terraform commands; for other names, `.auto` needs to be added before tfvars, such as `variables.auto.tfvars`)
2. Modify parameter values as needed
3. When executing `terraform plan` or `terraform apply`, Terraform will automatically read the variable values from this file

In addition to using `terraform.tfvars` file, variable values can also be set in the following ways:

1. Command line parameters: `terraform apply -var="vpc_name=my-vpc" -var="er_instance_name=my-er"`
2. Environment variables: `export TF_VAR_vpc_name=my-vpc`
3. Custom named variable files: `terraform apply -var-file="custom.tfvars"`

> Note: If the same variable is set in multiple ways, Terraform will use the variable value according to the following priority: command line parameters > variable files > environment variables > default values.

### 11. Initialize and Apply Terraform Configuration

After completing the above script configuration, execute the following steps to create resources:

1. Run `terraform init` to initialize the environment
2. Run `terraform plan` to view the resource creation plan
3. After confirming the resource plan is correct, run `terraform apply` to start creating flow logs
4. Run `terraform show` to view the created flow logs

## Reference Information

- [Huawei Cloud ER Product Documentation](https://support.huaweicloud.com/er/index.html)
- [Huawei Cloud LTS Product Documentation](https://support.huaweicloud.com/lts/index.html)
- [Huawei Cloud Provider Documentation](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs)
- [ER Best Practice Source Code Reference](https://github.com/huaweicloud/terraform-provider-huaweicloud/tree/master/examples/er)
