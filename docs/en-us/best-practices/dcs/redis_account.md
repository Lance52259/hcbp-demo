# Deploy Redis Account

## Application Scenario

Distributed Cache Service (DCS) is a high-performance, highly available in-memory database service provided by Huawei Cloud, supporting mainstream cache engines such as Redis and Memcached. DCS Redis instances support the creation of independent accounts for access control and permission management, ensuring data security.

This best practice will introduce how to use Terraform to automatically deploy a DCS Redis instance and create its account. Through this practice, you can quickly build a complete environment including a VPC, subnet, Redis instance, and account, and learn how to configure the account role (read/write permissions) and password management.

## Related Resources/Data Sources

This best practice involves the following main resources and data sources:

### Data Sources

- [Availability Zones (huaweicloud_availability_zones)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/availability_zones)
- [DCS Flavors (huaweicloud_dcs_flavors)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/dcs_flavors)

### Resources

- [Virtual Private Cloud (huaweicloud_vpc)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc)
- [Subnet (huaweicloud_vpc_subnet)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc_subnet)
- [Random Password (random_password)](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password)
- [DCS Redis Instance (huaweicloud_dcs_instance)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/dcs_instance)
- [DCS Account (huaweicloud_dcs_account)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/dcs_account)

### Resource/Data Source Dependencies

```
huaweicloud_vpc
    └── huaweicloud_vpc_subnet
huaweicloud_availability_zones (optional)
huaweicloud_dcs_flavors (optional)
random_password (optional)
huaweicloud_vpc_subnet
    └── huaweicloud_dcs_instance
huaweicloud_dcs_instance
    └── huaweicloud_dcs_account
```

## Operation Steps

### 1. Script Preparation

Prepare the TF file (such as main.tf) for writing the current best practice script in the specified working directory, and ensure that it (or other TF files in the same directory) contains the provider version declaration and Huawei Cloud authentication information required for deploying resources.
For configuration introduction, refer to [Preparation Before Deploying Huawei Cloud Resources](../../introductions/prepare_before_deploy.md).

### 2. Create VPC

Add the following script to the TF file (such as main.tf):

```hcl
# Create a VPC in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
variable "vpc_name" {
  description = "The name of the VPC"
  type        = string
}

variable "vpc_cidr" {
  description = "The CIDR block of the VPC"
  type        = string
  default     = "192.168.0.0/16"
}

resource "huaweicloud_vpc" "test" {
  name = var.vpc_name
  cidr = var.vpc_cidr
}
```

**Parameter Description**:
- **name**: Assigned by referencing the input variable vpc_name, specifying the name of the VPC.
- **cidr**: Assigned by referencing the input variable vpc_cidr, specifying the CIDR block of the VPC.

### 3. Create Subnet

Add the following script to the TF file (such as main.tf):

```hcl
# Create a subnet in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
variable "subnet_name" {
  description = "The name of the subnet"
  type        = string
}

variable "subnet_cidr" {
  description = "The CIDR block of the subnet"
  type        = string
  default     = ""
}

variable "subnet_gateway_ip" {
  description = "The gateway IP address of the subnet"
  type        = string
  default     = ""
}

resource "huaweicloud_vpc_subnet" "test" {
  vpc_id     = huaweicloud_vpc.test.id
  name       = var.subnet_name
  cidr       = var.subnet_cidr != "" ? var.subnet_cidr : cidrsubnet(huaweicloud_vpc.test.cidr, 8, 0)
  gateway_ip = (var.subnet_gateway_ip != "" ? var.subnet_gateway_ip :
  cidrhost(cidrsubnet(huaweicloud_vpc.test.cidr, 8, 0), 1))
}
```

**Parameter Description**:
- **vpc_id**: Assigned by referencing huaweicloud_vpc.test.id, specifying the ID of the VPC to which the subnet belongs.
- **name**: Assigned by referencing the input variable subnet_name, specifying the name of the subnet.
- **cidr**: Assigned by referencing the input variable subnet_cidr. When not set, a subnet is automatically divided from the VPC CIDR.
- **gateway_ip**: Assigned by referencing the input variable subnet_gateway_ip. When not set, the gateway IP is automatically calculated.

### 4. Query Availability Zones and DCS Flavors (Optional)

When the availability zone or instance flavor is not specified, you need to query the available availability zones and DCS flavors. Add the following script to the TF file (such as main.tf):

```hcl
# Query availability zones in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
data "huaweicloud_availability_zones" "test" {
  count = var.availability_zone == "" ? 1 : 0
}

# Query DCS flavors in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
data "huaweicloud_dcs_flavors" "test" {
  count = var.instance_flavor_id == "" ? 1 : 0

  cache_mode     = "ha"
  capacity       = var.instance_capacity
  engine_version = var.instance_engine_version
}
```

**Parameter Description**:
- **availability_zone**: Determined by referencing the input variable availability_zone. When not specified, the list of availability zones is queried.
- **instance_flavor_id**: Determined by referencing the input variable instance_flavor_id. When not specified, DCS flavors are queried.
- **cache_mode**: The cache engine mode, fixed to "ha".
- **capacity**: Assigned by referencing the input variable instance_capacity, specifying the cache capacity.
- **engine_version**: Assigned by referencing the input variable instance_engine_version, specifying the engine version.

### 5. Generate Random Password (Optional)

When the instance password is not specified, a random password needs to be generated. Add the following script to the TF file (such as main.tf):

```hcl
# Generate a random password
resource "random_password" "test" {
  count = var.instance_password == "" ? 1 : 0

  length           = 12
  special          = true
  override_special = "!@%^*-_=+"
  min_upper        = 1
  min_lower        = 1
  min_numeric      = 1
  min_special      = 1
}
```

**Parameter Description**:
- **length**: The length of the password, fixed to 12.
- **special**: Whether to include special characters, fixed to true.
- **override_special**: Specifies the set of special characters to use.
- **min_upper**: The minimum number of uppercase letters.
- **min_lower**: The minimum number of lowercase letters.
- **min_numeric**: The minimum number of numeric characters.
- **min_special**: The minimum number of special characters.

### 6. Create DCS Redis Instance

Add the following script to the TF file (such as main.tf):

```hcl
# Create a DCS Redis instance in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
variable "instance_name" {
  description = "The name of the Redis instance"
  type        = string
}

variable "instance_capacity" {
  description = "The capacity of the Redis instance (in GB)"
  type        = number
  default     = 1
}

variable "instance_engine_version" {
  description = "The engine version of the Redis instance"
  type        = string
  default     = "5.0"
}

variable "instance_password" {
  description = "The password for the Redis instance"
  type        = string
  sensitive   = true
  default     = null
}

variable "availability_zone" {
  description = "The availability zone to which the Redis instance belongs"
  type        = string
  default     = ""
}

variable "instance_flavor_id" {
  description = "The flavor ID of the Redis instance"
  type        = string
  default     = ""
}

resource "huaweicloud_dcs_instance" "test" {
  name               = var.instance_name
  engine             = "Redis"
  engine_version     = var.instance_engine_version
  capacity           = var.instance_capacity
  vpc_id             = huaweicloud_vpc.test.id
  subnet_id          = huaweicloud_vpc_subnet.test.id
  password           = (var.instance_password != "" ? var.instance_password :
  try(random_password.test[0].result, null))
  flavor             = (var.instance_flavor_id != "" ? var.instance_flavor_id :
  try(data.huaweicloud_dcs_flavors.test[0].flavors[0].name, null))
  availability_zones = (var.availability_zone != "" ? [var.availability_zone] :
  try(slice(data.huaweicloud_availability_zones.test[0].names, 0, 1), null))
}
```

**Parameter Description**:
- **name**: Assigned by referencing the input variable instance_name, specifying the name of the Redis instance.
- **engine**: The cache engine, fixed to "Redis".
- **engine_version**: Assigned by referencing the input variable instance_engine_version, specifying the engine version.
- **capacity**: Assigned by referencing the input variable instance_capacity, specifying the cache capacity (GB).
- **vpc_id**: Assigned by referencing huaweicloud_vpc.test.id, specifying the VPC to which the instance belongs.
- **subnet_id**: Assigned by referencing huaweicloud_vpc_subnet.test.id, specifying the subnet to which the instance belongs.
- **password**: Assigned by referencing the input variable instance_password. When not set, a random password is used.
- **flavor**: Assigned by referencing the input variable instance_flavor_id. When not set, the queried flavor is used.
- **availability_zones**: Assigned by referencing the input variable availability_zone. When not set, the queried availability zone is used.

### 7. Create DCS Account

Add the following script to the TF file (such as main.tf):

```hcl
# Create a DCS account in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
variable "account_name" {
  description = "The name of the DCS account"
  type        = string
}

variable "account_role" {
  description = "The role of the DCS account. Valid values: read, write"
  type        = string
  default     = "read"
}

variable "account_password" {
  description = "The password of the DCS account"
  type        = string
  sensitive   = true
}

variable "account_description" {
  description = "The description of the DCS account"
  type        = string
  default     = ""
}

resource "huaweicloud_dcs_account" "test" {
  instance_id      = huaweicloud_dcs_instance.test.id
  account_name     = var.account_name
  account_role     = var.account_role
  account_password = var.account_password
  description      = var.account_description
}
```

**Parameter Description**:
- **instance_id**: Assigned by referencing huaweicloud_dcs_instance.test.id, specifying the Redis instance to which the account belongs.
- **account_name**: Assigned by referencing the input variable account_name, specifying the account name.
- **account_role**: Assigned by referencing the input variable account_role, specifying the role of the account. Valid values are read (read-only) and write (read and write).
- **account_password**: Assigned by referencing the input variable account_password, specifying the password of the account.
- **description**: Assigned by referencing the input variable account_description, specifying the description of the account.

### 8. Preset Input Parameters Required for Resource Deployment (Optional)

In this practice, some resources and data sources use input variables to assign values to configurations. These input parameters need to be manually entered during subsequent deployment.
Meanwhile, Terraform provides a method to preset these configurations through `tfvars` files, avoiding repeated input during each execution.

Create a `terraform.tfvars` file in the working directory with the following example content:

```hcl
# Fill in according to the script variables; use placeholders for sensitive information
vpc_name         = "tf_test_dcs_instance_vpc2"
subnet_name      = "tf_test_dcs_instance_subnet"
instance_name    = "tf_test_dcs_instance"
account_name     = "tf_test_account"
account_password = "Terraform@123"
```

**Usage**:

1. Save the above content as the `terraform.tfvars` file in the working directory (this file name allows Terraform to automatically import the content of the `tfvars` file when executing terraform commands; other names need to add `.auto` before tfvars, such as `variables.auto.tfvars`)
2. Modify the parameter values as needed
3. When executing `terraform plan` or `terraform apply`, Terraform will automatically read the variable values from this file

In addition to using the `terraform.tfvars` file, you can also set variable values in the following ways:

1. Command line parameters: `terraform apply -var="vpc_name=my-vpc"`
2. Environment variables: `export TF_VAR_vpc_name=my-vpc`
3. Custom-named variable files: `terraform apply -var-file="custom.tfvars"`

> Note: If the same variable is set in multiple ways, Terraform will use the variable values in the following priority: command line parameters > variable files > environment variables > default values.

### 9. Initialize and Apply Terraform Configuration

After completing the above script configuration, execute the following steps to create resources:

1. Run `terraform init` to initialize the environment
2. Run `terraform plan` to view the resource creation plan
3. After confirming the resource plan is correct, run `terraform apply` to start creating the DCS Redis instance and account
4. Run `terraform show` to view the created DCS Redis instance and account

## Reference Information

- [Huawei Cloud DCS Product Documentation](https://support.huaweicloud.com/dcs/index.html)
- [Huawei Cloud Provider Documentation](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs)
- [Best Practice Source Code Reference For DCS Redis Account](https://github.com/huaweicloud/terraform-provider-huaweicloud/tree/master/examples/dcs/redis-account)
