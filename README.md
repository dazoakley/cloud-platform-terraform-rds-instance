# cloud-platform-terraform-rds-instance
The Terraform module will create an RDS instance, RDS subnet group and a KMS key in AWS with AWS credentials that will have access to the resources. 

The RDS instance created will have a name in the format of `"cloud-platform-${var.application}-${var.environment-name}-${random_string.identifier.result}"`. This ensures that the instance created is globally unique and can be recognised easily. e.g. `cloud-platform-laa-fee-calculator-staging-fjldkaw` 

The RDS instance is deployed in the `live-0` VPC. This is specified in the `aws_db_subnet_group` resource and the output is referenced in the `aws_db_instance`. This is also specified in the `vpc_security_group_ids` where the `live-0` security groups have been referenced. As the cloud platform grows, users will be able to specify the cluster they choose to deploy the RDS instance to e.g. `live-0`, `live-1`, `live-2` etc. The `aws_db_subnet_group` will be provisioned in the format of `"${var.application}-${var.environment-name}-db-subnet-group-${random_string.subnet.result}"`. 

The instance by default is encrypted and the AWS KMS key is provisioned and displayed as an output. The KMS key name is generated using a similar format of the other resources e.g. `"alias/${var.application}-${var.environment-name}-kms-key-${random_string.key.result}"`. 

The module also deploys the instance in Multi-AZ.

The database name uses the format - `"${var.application}${var.environment-name}"`. The username and password is generated by Terraform using a random string and is displayed as an output. 

## Usage

```hcl
module "example_team_rds" {
  source = "github.com/ministryofjustice/cloud-platform-terraform-rds-instance?ref=master"

  team_name              = "example-repo"
  db_allocated_storage   = 20
  db_engine              = "mysql"
  db_engine_version      = 5.7
  db_instance_class      = "db.t2.small"
  db_retention_period    = 10
  db_port                = 3306
  db_storage_type        = "io1"
  db_iops                = 1000
  business-unit          = "example-bu"
  application            = "example-app"
  is-production          = "false"
  environment-name       = "development"
  infrastructure-support = "example-team@digtal.justice.gov.uk"
}
```
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| db_allocated_storage | The allocated storage in gibibytes | integer | `10` | no |
| db_engine | Database engine used | string | `postgresql` | no |
| db_engine_version | The engine version to use | integer | `10.4` | no |
| db_instance_class | The instance type of the RDS instance | string | `db.t2.small` | no |
| db_backup_retention_period | The days to retain backups. Must be 1 or greater to be a source for a Read Replica | integer | - | yes
| db_port | The port on which the DB accepts connections | integer | - | no |
| db_storage_type | One of standard (magnetic), gp2 (general purpose SSD), or io1 (provisioned IOPS SSD). | string | `gp2` | no |
| db_iops | The amount of provisioned IOPS. Setting this implies a storage_type of io1 | integer | `0` | * Required if 'db_storage_type' is set to io1 |


### Tags

Some of the inputs are tags. All infrastructure resources need to be tagged according to MOJ techincal guidence. The tags are stored as variables that you will need to fill out as part of your module.

https://ministryofjustice.github.io/technical-guidance/standards/documenting-infrastructure-owners/#documenting-owners-of-infrastructure

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| application |  | string | - | yes |
| business-unit | Area of the MOJ responsible for the service | string | `mojdigital` | yes |
| environment-name |  | string | - | yes |
| infrastructure-support | The team responsible for managing the infrastructure. Should be of the form team-email | string | - | yes |
| is-production |  | string | `false` | yes |
| team_name |  | string | - | yes |

## Outputs

| Name | Description |
|------|-------------|
| access_key_id | Access key id for rds account |
| secret_access_key | Secret key for rds account |
| rds_instance_endpoint | The connection endpoint in address:port format |
| rds_instance_arn | The ARN of the RDS instance |
| database_name | Name of the database |
| database_username | Database Username |
| database_password | Database Password |
| database_subnet_group_arn | The ARN of the db subnet group |
| kms_key_arn | The Amazon Resource Name ARN of the key. |
| kms_key_id | The globally unique identifier for the KMS key. |


## Reading Material

- https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MySQL.html
- https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_PostgreSQL.html
- https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MariaDB.html
