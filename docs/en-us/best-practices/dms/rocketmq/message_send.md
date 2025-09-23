# Deploy RocketMQ Message Send

## Application Scenario

Huawei Cloud Distributed Message Service RocketMQ is a highly available, highly reliable, and high-performance distributed message middleware service, widely used in distributed systems in industries such as e-commerce, finance, and IoT.
Message sending is one of the core functions of RocketMQ, used to send business data as messages to specified topics for consumer processing.

Through RocketMQ message sending functionality, enterprises can implement modern application patterns such as asynchronous message processing, event-driven architecture, and system decoupling, meeting message communication requirements for different business scenarios.
This best practice will introduce how to use Terraform to automatically deploy RocketMQ message sending functionality, including RocketMQ instance, topic, and message sending creation.

## Related Resources/Data Sources

This best practice involves the following main resources and data sources:

### Data Sources

- [RocketMQ Availability Zones Query Data Source (data.huaweicloud_dms_rocketmq_availability_zones)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/dms_rocketmq_availability_zones)
- [RocketMQ Flavors Query Data Source (data.huaweicloud_dms_rocketmq_flavors)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/dms_rocketmq_flavors)
- [RocketMQ Broker Query Data Source (data.huaweicloud_dms_rocketmq_broker)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/dms_rocketmq_broker)

### Resources

- [VPC Resource (huaweicloud_vpc)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc)
- [VPC Subnet Resource (huaweicloud_vpc_subnet)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc_subnet)
- [Security Group Resource (huaweicloud_networking_secgroup)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/networking_secgroup)
- [RocketMQ Instance Resource (huaweicloud_dms_rocketmq_instance)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/dms_rocketmq_instance)
- [RocketMQ Topic Resource (huaweicloud_dms_rocketmq_topic)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/dms_rocketmq_topic)
- [RocketMQ Message Send Resource (huaweicloud_dms_rocketmq_message_send)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/dms_rocketmq_message_send)

### Resource/Data Source Dependencies

```text
data.huaweicloud_dms_rocketmq_availability_zones
    └── data.huaweicloud_dms_rocketmq_flavors
        └── huaweicloud_dms_rocketmq_instance
            └── data.huaweicloud_dms_rocketmq_broker
                └── huaweicloud_dms_rocketmq_topic
                    └── huaweicloud_dms_rocketmq_message_send

huaweicloud_vpc
    └── huaweicloud_vpc_subnet
        └── huaweicloud_dms_rocketmq_instance
            └── huaweicloud_dms_rocketmq_topic
                └── huaweicloud_dms_rocketmq_message_send

huaweicloud_networking_secgroup
    └── huaweicloud_dms_rocketmq_instance
        └── huaweicloud_dms_rocketmq_topic
            └── huaweicloud_dms_rocketmq_message_send
```

## Operation Steps

### 1. Script Preparation

Prepare the TF file (e.g., main.tf) in the specified workspace for writing the current best practice script, ensuring that it (or other TF files in the same directory) contains the provider version declaration and Huawei Cloud authentication information required for deploying resources.
Refer to the "Preparation Before Deploying Huawei Cloud Resources" document for configuration introduction.

### 2. Query RocketMQ Availability Zone Information Through Data Source

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to perform a data source query, the query results are used to create RocketMQ instances:

```hcl
variable "availability_zones" {
  description = "The availability zones to which the RocketMQ instance belongs"
  type        = list(string)
  default     = []
  nullable    = false
}

# Get all RocketMQ availability zone information under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block), used to create RocketMQ instances
data "huaweicloud_dms_rocketmq_availability_zones" "test" {
  count = length(var.availability_zones) == 0 ? 1 : 0
}
```

**Parameter Description**:

- **count**: Data source creation count, used to control whether to execute the availability zones list query data source, only creates data source when availability_zones is empty (i.e., executes availability zones list query)

### 3. Query RocketMQ Flavor Information Through Data Source

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to perform a data source query, the query results are used to create RocketMQ instances:

```hcl
variable "instance_flavor_id" {
  description = "The flavor ID of the RocketMQ instance"
  type        = string
  default     = ""
  nullable    = false
}

variable "instance_flavor_type" {
  description = "The flavor type of the RocketMQ instance"
  type        = string
  default     = "cluster.small"
}

variable "availability_zones_count" {
  description = "The number of availability zones"
  type        = number
  default     = 1
}

# Get all RocketMQ flavor information under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block), used to create RocketMQ instances
data "huaweicloud_dms_rocketmq_flavors" "test" {
  count = var.instance_flavor_id == "" ? 1 : 0

  type               = var.instance_flavor_type
  availability_zones = length(var.availability_zones) != 0 ? var.availability_zones : slice(data.huaweicloud_dms_rocketmq_availability_zones.test[0].availability_zones[*].code, 0, var.availability_zones_count)
}
```

**Parameter Description**:

- **count**: Data source creation count, used to control whether to execute the flavors list query data source, only creates data source when `var.instance_flavor_id` is empty (i.e., executes flavors list query)
- **type**: RocketMQ instance flavor type, prioritizes using the flavor type specified in input variables, defaults to "cluster.small" if not specified
- **availability_zones**: Availability zones list, prioritizes using the availability zones specified in input variables, uses the first `var.availability_zones_count` availability zones from data source query results if not specified
- **availability_zones_count**: Number of availability zones, prioritizes using the availability zones count specified in input variables, defaults to 1 if not specified

### 4. Create VPC Resource

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to create a VPC resource:

```hcl
variable "vpc_name" {
  description = "The VPC name"
  type        = string
}

variable "vpc_cidr" {
  description = "The CIDR block of the VPC"
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

- **name**: VPC name
- **cidr**: VPC CIDR block, prioritizes using the CIDR block specified in input variables, defaults to "192.168.0.0/16" if not specified

### 5. Create VPC Subnet Resource

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to create a VPC subnet resource:

```hcl
variable "subnet_name" {
  description = "The subnet name"
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

# Create a VPC subnet resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_vpc_subnet" "test" {
  vpc_id     = huaweicloud_vpc.test.id
  name       = var.subnet_name
  cidr       = var.subnet_cidr == "" ? cidrsubnet(huaweicloud_vpc.test.cidr, 8, 0) : var.subnet_cidr
  gateway_ip = var.subnet_gateway_ip == "" ? cidrhost(cidrsubnet(huaweicloud_vpc.test.cidr, 8, 0), 1) : var.subnet_gateway_ip
}
```

**Parameter Description**:

- **vpc_id**: Subnet's VPC ID, references the ID of the VPC resource created earlier
- **name**: Subnet name
- **cidr**: Subnet CIDR block, prioritizes using the CIDR block specified in input variables, automatically calculated if not specified
- **gateway_ip**: Subnet gateway IP, prioritizes using the gateway IP specified in input variables, automatically calculated if not specified

### 6. Create Security Group Resource

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to create a security group resource:

```hcl
variable "security_group_name" {
  description = "The security group name"
  type        = string
}

# Create a security group resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_networking_secgroup" "test" {
  name                 = var.security_group_name
  delete_default_rules = true
}
```

**Parameter Description**:

- **name**: Security group name
- **delete_default_rules**: Whether to delete default rules, set to true to delete default rules

### 7. Create RocketMQ Instance

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to create a RocketMQ instance resource:

```hcl
variable "instance_name" {
  description = "The name of the RocketMQ instance"
  type        = string
}

variable "instance_engine_version" {
  description = "The engine version of the RocketMQ instance"
  type        = string
  default     = ""
  nullable    = false

  validation {
    condition     = var.instance_flavor_id == "" || var.instance_engine_version != ""
    error_message = "When 'instance_flavor_id' is not empty, 'instance_engine_version' is required"
  }
}

variable "instance_storage_spec_code" {
  description = "The storage spec code of the RocketMQ instance"
  type        = string
  default     = ""
  nullable    = false

  validation {
    condition     = var.instance_flavor_id == "" || var.instance_storage_spec_code != ""
    error_message = "When 'instance_flavor_id' is not empty, 'instance_storage_spec_code' is required"
  }
}

variable "instance_storage_space" {
  description = "The storage space of the RocketMQ instance in GB"
  type        = number
  default     = 800
}

variable "instance_broker_num" {
  description = "The number of brokers for the RocketMQ instance"
  type        = number
  default     = 1
}

variable "instance_description" {
  description = "The description of the RocketMQ instance"
  type        = string
  default     = ""
}

variable "instance_tags" {
  description = "The tags of the RocketMQ instance"
  type        = map(string)
  default     = {}
}

variable "enterprise_project_id" {
  description = "The enterprise project ID"
  type        = string
  default     = null
}

variable "instance_enable_acl" {
  description = "Whether to enable ACL for the RocketMQ instance"
  type        = bool
  default     = false
}

variable "instance_tls_mode" {
  description = "The TLS mode of the RocketMQ instance"
  type        = string
  default     = "SSL"
}

variable "instance_configs" {
  description = "The configuration parameters of the RocketMQ instance"
  type = list(object({
    name  = string
    value = string
  }))

  nullable = false
  default  = []
}

# Create a RocketMQ instance resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_dms_rocketmq_instance" "test" {
  name                  = var.instance_name
  flavor_id             = var.instance_flavor_id != "" ? var.instance_flavor_id : try(data.huaweicloud_dms_rocketmq_flavors.test[0].flavors[0].id, null)
  engine_version        = var.instance_engine_version != "" ? var.instance_engine_version : try(data.huaweicloud_dms_rocketmq_flavors.test[0].versions[0], null)
  storage_spec_code     = var.instance_storage_spec_code != "" ? var.instance_storage_spec_code : try(data.huaweicloud_dms_rocketmq_flavors.test[0].flavors[0].ios[0].storage_spec_code, null)
  storage_space         = var.instance_storage_space
  availability_zones    = length(var.availability_zones) != 0 ? var.availability_zones : slice(data.huaweicloud_dms_rocketmq_availability_zones.test[0].availability_zones[*].code, 0, var.availability_zones_count)
  vpc_id                = huaweicloud_vpc.test.id
  subnet_id             = huaweicloud_vpc_subnet.test.id
  security_group_id     = huaweicloud_networking_secgroup.test.id
  broker_num            = var.instance_broker_num
  description           = var.instance_description
  tags                  = var.instance_tags
  enterprise_project_id = var.enterprise_project_id
  enable_acl            = var.instance_enable_acl
  tls_mode              = var.instance_tls_mode

  dynamic "configs" {
    for_each = var.instance_configs

    content {
      name  = configs.value.name
      value = configs.value.value
    }
  }
}
```

**Parameter Description**:

- **name**: RocketMQ instance name
- **flavor_id**: RocketMQ instance flavor ID, prioritizes using the flavor specified in input variables, uses data source query results if not specified
- **engine_version**: RocketMQ engine version, prioritizes using the version specified in input variables, uses data source query results if not specified
- **storage_spec_code**: Storage spec code, prioritizes using the spec specified in input variables, uses data source query results if not specified
- **storage_space**: Storage space size, prioritizes using the storage space size specified in input variables, defaults to 800GB if not specified
- **availability_zones**: Availability zones list, prioritizes using the availability zones specified in input variables, uses data source query results if not specified
- **vpc_id**: VPC ID, references the ID of the VPC resource created earlier
- **subnet_id**: Subnet ID, references the ID of the subnet resource created earlier
- **security_group_id**: Security group ID, references the ID of the security group resource created earlier
- **broker_num**: Broker count, prioritizes using the broker count specified in input variables, defaults to 1 if not specified
- **description**: Instance description
- **tags**: Instance tags
- **enterprise_project_id**: Enterprise project ID
- **enable_acl**: Whether to enable ACL, prioritizes using the enable ACL setting specified in input variables, defaults to false if not specified
- **tls_mode**: TLS mode, prioritizes using the TLS mode specified in input variables, defaults to "SSL" if not specified
- **configs**: Instance configuration block

### 8. Query RocketMQ Broker Information Through Data Source

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to perform a data source query, the query results are used to create topics:

```hcl
variable "topic_brokers" {
  description = "The broker list of the topic, it's only valid when the RocketMQ instance version is `4.8.0`"
  type        = list(string)
  default     = []
  nullable    = false
}

# Get all RocketMQ Broker information under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block), used to create topics
data "huaweicloud_dms_rocketmq_broker" "test" {
  count = length(var.topic_brokers) == 0 && huaweicloud_dms_rocketmq_instance.test.engine_version == "4.8.0" ? 1 : 0

  instance_id = huaweicloud_dms_rocketmq_instance.test.id
}
```

**Parameter Description**:

- **count**: Data source creation count, used to control whether to execute the Broker query data source, only creates data source when topic broker list is not specified and RocketMQ instance version is 4.8.0
- **instance_id**: RocketMQ instance ID, references the ID of the RocketMQ instance resource created earlier

### 9. Create RocketMQ Topic

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to create a RocketMQ topic resource:

```hcl
variable "topic_name" {
  description = "The name of the RocketMQ topic"
  type        = string
}

variable "topic_message_type" {
  description = "The message type of the RocketMQ topic"
  type        = string
  default     = "NORMAL"
}

variable "topic_queue_num" {
  description = "The queue number of the RocketMQ topic"
  type        = number
  default     = 3
}

variable "topic_permission" {
  description = "The permission of the RocketMQ topic"
  type        = string
  default     = "all"
}

# Create a RocketMQ topic resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_dms_rocketmq_topic" "test" {
  instance_id  = huaweicloud_dms_rocketmq_instance.test.id
  name         = var.topic_name
  message_type = var.topic_message_type
  queue_num    = var.topic_queue_num
  permission   = var.topic_permission

  dynamic "brokers" {
    for_each = length(var.topic_brokers) > 0 ? var.topic_brokers : try(data.huaweicloud_dms_rocketmq_broker.test[0].brokers, [])

    content {
      name = brokers.value
    }
  }
}
```

**Parameter Description**:

- **instance_id**: RocketMQ instance ID, references the ID of the RocketMQ instance resource created earlier
- **name**: Topic name
- **message_type**: Message type, prioritizes using the message type specified in input variables, defaults to "NORMAL" if not specified, only valid when RocketMQ instance version is 5.x
- **queue_num**: Queue count, prioritizes using the queue count specified in input variables, defaults to 3 if not specified, only valid when RocketMQ instance version is 4.8.0
- **permission**: Topic permission, prioritizes using the topic permission specified in input variables, defaults to "all" if not specified, only valid when RocketMQ instance version is 4.8.0
- **brokers**: Broker configuration block, prioritizes using the broker list specified in input variables, uses Broker query data source results if not specified

### 10. Create RocketMQ Message Send

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to create a RocketMQ message send resource:

```hcl
variable "message_body" {
  description = "The body content of the message"
  type        = string
}

variable "message_properties" {
  description = "The property list of the message"
  type = list(object({
    name  = string
    value = string
  }))
  default  = []
  nullable = false
}

# Create a RocketMQ message send resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_dms_rocketmq_message_send" "test" {
  instance_id = huaweicloud_dms_rocketmq_instance.test.id
  topic       = huaweicloud_dms_rocketmq_topic.test.name
  body        = var.message_body

  dynamic "property_list" {
    for_each = var.message_properties

    content {
      name  = property_list.value.name
      value = property_list.value.value
    }
  }
}
```

**Parameter Description**:

- **instance_id**: RocketMQ instance ID, references the ID of the RocketMQ instance resource created earlier
- **topic**: Topic name, references the name of the topic resource created earlier
- **body**: Message body content
- **property_list**: Message property list, used to set key-value pair properties for messages

### 11. Preset Input Parameters Required for Resource Deployment (Optional)

In this practice, some resources and data sources use input variables to assign values to configuration content. These input parameters need to be manually entered during subsequent deployments.
At the same time, Terraform provides a method to preset these configurations through `.tfvars` files, which can avoid repeated input during each execution.

Create a `terraform.tfvars` file in the working directory with the following example content:

```hcl
# VPC and subnet configuration
vpc_name            = "tf_test_vpc"
subnet_name         = "tf_test_subnet"
security_group_name = "tf_test_security_group"

# RocketMQ instance basic information
instance_name        = "tf_test_instance"
instance_broker_num  = 1
instance_description = "Created by terraform script"

# Topic basic information
topic_name       = "tf_test_topic"
topic_queue_num  = 3
topic_permission = "all"

# Message send configuration
message_body = "tf terraform script test"

message_properties = [
  {
    name  = "KEYS"
    value = "owner"
  },
  {
    name  = "TAGS"
    value = "terraform"
  }
]
```

**Usage**:

1. Save the above content as `terraform.tfvars` file in the working directory (this file name allows users to automatically import the content of this `tfvars` file when executing terraform commands; for other names, `.auto` needs to be added before tfvars, such as `variables.auto.tfvars`)
2. Modify parameter values as needed
3. When executing `terraform plan` or `terraform apply`, Terraform will automatically read the variable values from this file

In addition to using `terraform.tfvars` file, variable values can also be set in the following ways:

1. Command line parameters: `terraform apply -var="vpc_name=my-vpc" -var="subnet_name=my-subnet"`
2. Environment variables: `export TF_VAR_vpc_name=my-vpc`
3. Custom named variable files: `terraform apply -var-file="custom.tfvars"`

> Note: If the same variable is set in multiple ways, Terraform will use the variable value according to the following priority: command line parameters > variable files > environment variables > default values.

### 12. Initialize and Apply Terraform Configuration

After completing the above script configuration, execute the following steps to create resources:

1. Run `terraform init` to initialize the environment
2. Run `terraform plan` to view the resource creation plan
3. After confirming the resource plan is correct, run `terraform apply` to start creating the RocketMQ message send functionality
4. Run `terraform show` to view the details of the created RocketMQ message send functionality

## Reference Information

- [Huawei Cloud Distributed Message Service RocketMQ Product Documentation](https://support.huaweicloud.com/hrm/index.html)
- [Huawei Cloud Provider Documentation](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs)
- [RocketMQ Message Send Best Practice Source Code Reference](https://github.com/huaweicloud/terraform-provider-huaweicloud/tree/master/examples/dms/rocketmq/message-send)
