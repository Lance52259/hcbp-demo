# Deploy DDM Account

## Application Scenario

Distributed Database Middleware (DDM) is a distributed relational database middleware provided by Huawei Cloud. It is compatible with the MySQL protocol and supports capabilities such as database and table sharding and read/write splitting. In real business, you may need to create independent database accounts for DDM instances to access and manage logical databases.

This best practice will introduce how to use Terraform to automatically deploy a DDM instance and account, including creating VPC, subnet, and security group, configuring engine, flavor, and node number, and creating a DDM account with specified permissions.

## Related Resources/Data Sources

This best practice involves the following main resources and data sources:

### Data Sources

- [Availability Zones (data.huaweicloud_availability_zones)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/availability_zones)
- [DDM Engines (data.huaweicloud_ddm_engines)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/ddm_engines)
- [DDM Flavors (data.huaweicloud_ddm_flavors)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/ddm_flavors)

### Resources

- [Virtual Private Cloud (huaweicloud_vpc)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc)
- [Subnet (huaweicloud_vpc_subnet)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc_subnet)
- [Security Group (huaweicloud_networking_secgroup)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/networking_secgroup)
- [DDM Instance (huaweicloud_ddm_instance)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/ddm_instance)
- [Random Password (random_password)](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password)
- [DDM Account (huaweicloud_ddm_account)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/ddm_account)

### Resource/Data Source Dependencies

```
data.huaweicloud_availability_zones
    └── huaweicloud_ddm_instance

data.huaweicloud_ddm_engines
    └── data.huaweicloud_ddm_flavors
        └── huaweicloud_ddm_instance

huaweicloud_vpc
    └── huaweicloud_vpc_subnet
        └── huaweicloud_ddm_instance

huaweicloud_networking_secgroup
    └── huaweicloud_ddm_instance

huaweicloud_ddm_instance
    └── huaweicloud_ddm_account

random_password
    └── huaweicloud_ddm_account
```

## Operation Steps

### 1. Script Preparation

Prepare the TF file (such as main.tf) for writing the current best practice script in the specified working directory, and ensure that it (or other TF files in the same level directory) contains the provider version declaration and Huawei Cloud authentication information required for deploying resources.
For configuration introduction, refer to [Preparation Before Deploying Huawei Cloud Resources](../../introductions/prepare_before_deploy.md).

### 2. Query Availability Zones

Add the following script to the TF file (such as main.tf) to query available availability zones when no availability zone is specified:

```hcl
# Query availability zones in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
data "huaweicloud_availability_zones" "test" {
  count = length(var.availability_zones) == 0 ? 1 : 0
}
```

**Parameter Description**:
- **count**: When the input variable availability_zones is an empty list, create this data source to query availability zones.

### 3. Create VPC

Add the following script to the TF file (such as main.tf) to create a VPC:

```hcl
# Create VPC in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
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
- **name**: Assigned by referencing the input variable vpc_name.
- **cidr**: Assigned by referencing the input variable vpc_cidr, with a default value of "192.168.0.0/16".

### 4. Create Subnet

Add the following script to the TF file (such as main.tf) to create a subnet:

```hcl
# Create subnet in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
variable "subnet_name" {
  description = "The name of the subnet"
  type        = string
}

variable "subnet_cidr" {
  description = "The CIDR block of the subnet"
  type        = string
  default     = ""
  nullable    = false
}

variable "subnet_gateway_ip" {
  description = "The gateway IP of the subnet"
  type        = string
  default     = ""
  nullable    = false
}

resource "huaweicloud_vpc_subnet" "test" {
  vpc_id     = huaweicloud_vpc.test.id
  name       = var.subnet_name
  cidr       = var.subnet_cidr == "" ? cidrsubnet(huaweicloud_vpc.test.cidr, 8, 0) : var.subnet_cidr
  gateway_ip = var.subnet_gateway_ip == "" ? cidrhost(cidrsubnet(huaweicloud_vpc.test.cidr, 8, 0), 1) : var.subnet_gateway_ip
}
```

**Parameter Description**:
- **vpc_id**: Assigned by referencing the ID of huaweicloud_vpc.test.
- **name**: Assigned by referencing the input variable subnet_name.
- **cidr**: Assigned by referencing the input variable subnet_cidr; if not specified, it is automatically divided from the VPC CIDR.
- **gateway_ip**: Assigned by referencing the input variable subnet_gateway_ip; if not specified, the gateway address is automatically calculated.

### 5. Create Security Group

Add the following script to the TF file (such as main.tf) to create a security group:

```hcl
# Create security group in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
variable "security_group_name" {
  description = "The name of the security group"
  type        = string
}

resource "huaweicloud_networking_secgroup" "test" {
  name                 = var.security_group_name
  delete_default_rules = true
}
```

**Parameter Description**:
- **name**: Assigned by referencing the input variable security_group_name.
- **delete_default_rules**: Set to true to delete default security group rules.

### 6. Query DDM Engines and Flavors

Add the following script to the TF file (such as main.tf) to query available DDM engines and flavors when no engine and flavor are specified:

```hcl
# Query DDM engines in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
data "huaweicloud_ddm_engines" "test" {
  count = var.instance_engine_id == "" ? 1 : 0
}

# Query DDM flavors in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
data "huaweicloud_ddm_flavors" "test" {
  count = var.instance_flavor_id == "" ? 1 : 0

  engine_id = var.instance_engine_id == "" ? try(data.huaweicloud_ddm_engines.test[0].engines[0].id, null) : var.instance_engine_id
}
```

**Parameter Description**:
- **count**: When the input variable instance_engine_id or instance_flavor_id is empty, create the data source to query the default engine and flavor.
- **engine_id**: Assigned by referencing the input variable instance_engine_id or the query result of data.huaweicloud_ddm_engines.test.

### 7. Create DDM Instance

Add the following script to the TF file (such as main.tf) to create a DDM instance:

```hcl
# Create DDM instance in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
variable "availability_zones" {
  description = "The availability zones to which the DDM instance belongs"
  type        = list(string)
  default     = []
  nullable    = false
}

variable "instance_engine_id" {
  description = "The engine ID of the DDM instance"
  type        = string
  default     = ""
  nullable    = false
}

variable "instance_flavor_id" {
  description = "The flavor ID of the DDM instance"
  type        = string
  default     = ""
  nullable    = false
}

variable "instance_name" {
  description = "The name of the DDM instance"
  type        = string
}

variable "instance_node_num" {
  description = "The number of nodes in the DDM instance"
  type        = number
  default     = 2
}

variable "instance_parameters" {
  description = "The parameters of the DDM instance"

  type = list(object({
    name  = string
    value = string
  }))

  default = []
}

resource "huaweicloud_ddm_instance" "test" {
  name               = var.instance_name
  availability_zones = length(var.availability_zones) == 0 ? try(slice(data.huaweicloud_availability_zones.test[0].names, 0, 1)) : var.availability_zones
  engine_id          = var.instance_engine_id == "" ? try(data.huaweicloud_ddm_engines.test[0].engines[0].id, null) : var.instance_engine_id
  flavor_id          = var.instance_flavor_id == "" ? try(data.huaweicloud_ddm_flavors.test[0].flavors[0].id, null) : var.instance_flavor_id
  vpc_id             = huaweicloud_vpc.test.id
  subnet_id          = huaweicloud_vpc_subnet.test.id
  security_group_id  = huaweicloud_networking_secgroup.test.id
  node_num           = var.instance_node_num

  dynamic "parameters" {
    for_each = var.instance_parameters

    content {
      name  = parameters.value.name
      value = parameters.value.value
    }
  }
}
```

**Parameter Description**:
- **name**: Assigned by referencing the input variable instance_name.
- **availability_zones**: Assigned by referencing the input variable availability_zones or the query result of data.huaweicloud_availability_zones.test.
- **engine_id**: Assigned by referencing the input variable instance_engine_id or the query result of data.huaweicloud_ddm_engines.test.
- **flavor_id**: Assigned by referencing the input variable instance_flavor_id or the query result of data.huaweicloud_ddm_flavors.test.
- **vpc_id**: Assigned by referencing the ID of huaweicloud_vpc.test.
- **subnet_id**: Assigned by referencing the ID of huaweicloud_vpc_subnet.test.
- **security_group_id**: Assigned by referencing the ID of huaweicloud_networking_secgroup.test.
- **node_num**: Assigned by referencing the input variable instance_node_num, with a default value of 2.
- **parameters**: Dynamically set instance parameters by referencing the input variable instance_parameters.

### 8. Generate Random Password

Add the following script to the TF file (such as main.tf) to generate a random password when no account password is specified:

```hcl
# Generate random password
variable "account_password" {
  description = "The password of the DDM account"
  sensitive   = true
  type        = string
  default     = ""
  nullable    = false
}

resource "random_password" "test" {
  count = var.account_password == "" ? 1 : 0

  length           = 12
  special          = true
  override_special = "!@#%^*-_+?"
  min_upper        = 1
  min_lower        = 1
  min_numeric      = 1
  min_special      = 1
}
```

**Parameter Description**:
- **count**: When the input variable account_password is empty, create this resource to generate a random password.
- **length**: The password length is 12.
- **special**: Include special characters.
- **override_special**: Specify the set of allowed special characters.
- **min_upper**: At least 1 uppercase letter.
- **min_lower**: At least 1 lowercase letter.
- **min_numeric**: At least 1 digit.
- **min_special**: At least 1 special character.

### 9. Create DDM Account

Add the following script to the TF file (such as main.tf) to create a DDM account:

```hcl
# Create DDM account in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
variable "account_name" {
  description = "The name of the DDM account"
  type        = string
}

variable "account_permissions" {
  description = "The basic permissions of the DDM account"
  type        = list(string)
  default     = ["SELECT"]
  nullable    = false
}

variable "account_description" {
  description = "The description of the DDM account"
  type        = string
  default     = ""
}

resource "huaweicloud_ddm_account" "test" {
  instance_id = huaweicloud_ddm_instance.test.id
  name        = var.account_name
  password    = var.account_password == "" ? try(random_password.test[0].result, null) : var.account_password
  permissions = var.account_permissions
  description = var.account_description
}
```

**Parameter Description**:
- **instance_id**: Assigned by referencing the ID of huaweicloud_ddm_instance.test.
- **name**: Assigned by referencing the input variable account_name.
- **password**: Assigned by referencing the input variable account_password or the generated result of random_password.test.
- **permissions**: Assigned by referencing the input variable account_permissions, with a default value of ["SELECT"].
- **description**: Assigned by referencing the input variable account_description.

### 10. Preset Input Parameters Required for Resource Deployment (Optional)

In this practice, some resources and data sources use input variables to assign values to configuration content. These input parameters need to be manually entered during subsequent deployment.
Meanwhile, Terraform provides a method to preset these configurations through `tfvars` files, which can avoid repeated input during each execution.

Create a `terraform.tfvars` file in the working directory. The example content is as follows:

```hcl
# Fill in according to the script variables; use placeholders for sensitive information
vpc_name            = "your_vpc_name"
subnet_name         = "your_subnet_name"
security_group_name = "your_security_group_name"
instance_name       = "your_instance_name"
account_name        = "your_account_name"
```

**Usage**:

1. Save the above content as the `terraform.tfvars` file in the working directory (this file name allows Terraform to automatically import the content of the `tfvars` file when executing terraform commands; other names need to add `.auto` before tfvars, such as `variables.auto.tfvars`)
2. Modify the parameter values according to actual needs
3. When executing `terraform plan` or `terraform apply`, Terraform will automatically read the variable values from this file

In addition to using the `terraform.tfvars` file, you can also set variable values in the following ways:

1. Command line parameters: `terraform apply -var="vpc_name=my-vpc"`
2. Environment variables: `export TF_VAR_vpc_name=my-vpc`
3. Custom-named variable files: `terraform apply -var-file="custom.tfvars"`

> Note: If the same variable is set in multiple ways, Terraform will use the variable value according to the following priority: command line parameters > variable files > environment variables > default values.

### 11. Initialize and Apply Terraform Configuration

After completing the above script configuration, execute the following steps to create resources:

1. Run `terraform init` to initialize the environment
2. Run `terraform plan` to view the resource creation plan
3. After confirming the resource plan is correct, run `terraform apply` to start creating the DDM instance and account
4. Run `terraform show` to view the created DDM instance and account

## Reference Information

- [Huawei Cloud DDM Product Documentation](https://support.huaweicloud.com/ddm/index.html)
- [Huawei Cloud Provider Documentation](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs)
- [Best Practice Source Code Reference For DDM Account](https://github.com/huaweicloud/terraform-provider-huaweicloud/tree/master/examples/ddm/ddm-account)
