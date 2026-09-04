# Deploy CPTS Project and Test Task

## Application Scenario

Cloud Performance Test Service (CPTS) is a performance testing service provided by Huawei Cloud, helping users perform stress tests on cloud applications to evaluate system performance under high concurrency. With CPTS, users can create test projects, configure test tasks, and simulate a large number of concurrent users to discover system bottlenecks and performance issues.

This best practice will introduce how to use Terraform to create a CPTS project and a test task. Through this practice, you will learn how to use Infrastructure as Code (IaC) to automate the deployment of CPTS projects and test tasks, laying a foundation for subsequent performance testing work.

## Related Resources/Data Sources

This best practice involves the following main resources:

### Resources

- [CPTS Project (huaweicloud_cpts_project)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/cpts_project)
- [CPTS Test Task (huaweicloud_cpts_task)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/cpts_task)

### Resource/Data Source Dependencies

```
huaweicloud_cpts_project
    └── huaweicloud_cpts_task
```

## Operation Steps

### 1. Script Preparation

Prepare the TF file (such as main.tf) for writing the best practice script in the specified workspace, and ensure that it (or other TF files in the same directory) contains the required provider version declaration and Huawei Cloud authentication information.
For configuration introduction, refer to [Preparation Before Deploying Huawei Cloud Resources](../../introductions/prepare_before_deploy.md).

### 2. Create a CPTS Project

Add the following script to the TF file (such as main.tf):

```hcl
# Create a CPTS project in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
variable "cpts_project_name" {
  description = "The name of the CPTS project"
  type        = string
}

variable "cpts_project_description" {
  description = "The description of the CPTS project"
  type        = string
  default     = ""
}

resource "huaweicloud_cpts_project" "test" {
  name        = var.cpts_project_name
  description = var.cpts_project_description
}
```

**Parameter Description**:
- **name**: Assigned by referencing the input variable cpts_project_name, indicating the name of the CPTS project.
- **description**: Assigned by referencing the input variable cpts_project_description, indicating the description of the CPTS project.

### 3. Create a CPTS Test Task

Add the following script to the TF file (such as main.tf):

```hcl
# Create a CPTS test task in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
variable "cpts_task_name" {
  description = "The name of the CPTS test task"
  type        = string
}

variable "cpts_task_benchmark_concurrency" {
  description = "The benchmark concurrency of the CPTS test task"
  type        = number
  default     = 200
}

resource "huaweicloud_cpts_task" "test" {
  name                  = var.cpts_task_name
  project_id            = huaweicloud_cpts_project.test.id
  benchmark_concurrency = var.cpts_task_benchmark_concurrency
}
```

**Parameter Description**:
- **name**: Assigned by referencing the input variable cpts_task_name, indicating the name of the CPTS test task.
- **project_id**: Assigned by referencing huaweicloud_cpts_project.test.id, indicating the ID of the CPTS project to which the test task belongs.
- **benchmark_concurrency**: Assigned by referencing the input variable cpts_task_benchmark_concurrency, indicating the benchmark concurrency of the CPTS test task.

### 4. Preset Input Parameters Required for Resource Deployment (Optional)

In this practice, some resources and data sources use input variables to assign values to configuration content. These input parameters need to be manually entered during subsequent deployment.
Meanwhile, Terraform provides a way to preset these configurations through `tfvars` files, which can avoid repeated input during each execution.

Create a `terraform.tfvars` file in the working directory with the following example content:

```hcl
# Fill in according to the script variables; use placeholders for sensitive information
cpts_project_name               = "tf_test_cpts_project"
cpts_project_description        = "Created by Terraform for CPTS best practice"
cpts_task_name                  = "tf_test_cpts_task"
cpts_task_benchmark_concurrency = 200
```

**Usage**:

1. Save the above content as the `terraform.tfvars` file in the working directory (this file name allows Terraform to automatically import the content of the `tfvars` file when executing terraform commands; other names need to add `.auto` before tfvars, such as `variables.auto.tfvars`)
2. Modify the parameter values as needed
3. When executing `terraform plan` or `terraform apply`, Terraform will automatically read the variable values from this file

In addition to using the `terraform.tfvars` file, you can also set variable values in the following ways:

1. Command line parameters: `terraform apply -var="cpts_project_name=my-project"`
2. Environment variables: `export TF_VAR_cpts_project_name=my-project`
3. Custom named variable files: `terraform apply -var-file="custom.tfvars"`

> Note: If the same variable is set in multiple ways, Terraform will use the variable value according to the following priority: command line parameters > variable files > environment variables > default values.

### 5. Initialize and Apply Terraform Configuration

After completing the above script configuration, execute the following steps to create resources:

1. Run `terraform init` to initialize the environment
2. Run `terraform plan` to view the resource creation plan
3. After confirming the resource plan is correct, run `terraform apply` to start creating the CPTS project and test task
4. Run `terraform show` to view the created CPTS project and test task

## Reference Information

- [Huawei Cloud Cloud Performance Test Service Product Documentation](https://support.huaweicloud.com/cpts/index.html)
- [Huawei Cloud Provider Documentation](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs)
- [Best Practice Source Code Reference For CPTS Project and Test Task](https://github.com/huaweicloud/terraform-provider-huaweicloud/tree/master/examples/cpts/project-with-task)
