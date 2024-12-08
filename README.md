# AWS SSO Permission Sets and SSO Account Assignments Modules

This repository contains two modules:

1. [Permission Sets Module](./permissions_sets/): This module creates a collection of [AWS SSO permission sets](https://docs.aws.amazon.com/singlesignon/latest/userguide/permissionsetsconcept.html). A permission set is a collection of administrator-defined policies that AWS SSO uses to determine a user's effective permissions to access a given AWS account. Permission sets can contain either AWS managed policies or custom policies stored in AWS SSO. These policies are documents that act as containers for one or more permission statements, representing individual access controls (allow or deny) for various tasks within the AWS account.

2. [Account Assignments Module](./accounts_assigning/): This module assigns [AWS SSO permission sets](https://docs.aws.amazon.com/singlesignon/latest/userguide/permissionsetsconcept.html) to users and groups from the [AWS SSO Identity Source](https://docs.aws.amazon.com/singlesignon/latest/userguide/manage-your-identity-source.html).

## Usage

### Permission Sets Module

```hcl
module "permission_sets" {
  source = "https://github.com/seifrajhi/aws-sso-terraform.git//permission-sets?ref=main"

  permission_sets = [
    {
      name               = "AdministratorAccess",
      description        = "Allow Full Access to the account",
      relay_state        = "",
      session_duration   = "",
      tags               = {},
      inline_policy      = "",
      policy_attachments = ["arn:aws:iam::aws:policy/AdministratorAccess"]
      customer_managed_policy_attachments = [{
        name = aws_iam_policy.S3Access.name
        path = aws_iam_policy.S3Access.path
      }]
    },
    {
      name                                = "S3AdministratorAccess",
      description                         = "Allow Full S3 Administrator access to the account",
      relay_state                         = "",
      session_duration                    = "",
      tags                                = {},
      inline_policy                       = data.aws_iam_policy_document.S3Access.json,
      policy_attachments                  = []
      customer_managed_policy_attachments = []
    }
  ]
  context = module.this.context
}

data "aws_iam_policy_document" "S3Access" {
  statement {
    sid = "1"
    actions = ["*"]
    resources = ["arn:aws:s3:::*"]
  }
}

resource "aws_iam_policy" "S3Access" {
  name   = "S3Access"
  path   = "/"
  policy = data.aws_iam_policy_document.S3Access.json
  tags   = module.this.tags
}
```

### Account Assignments Module

```hcl
module "sso_account_assignments" {
  source = "https://github.com/seifrajhi/aws-sso-terraform.git//account-assignments?ref=main"

  account_assignments = [
    {
      account = "123456789",
      permission_set_arn = "arn:aws:sso:::permissionSet/ssoins-0000000000000000/ps-31d20e5987f0ce66",
      permission_set_name = "Administrators",
      principal_type = "GROUP",
      principal_name = "Administrators"
    },
    {
      account = "123456789",
      permission_set_arn = "arn:aws:sso:::permissionSet/ssoins-0000000000000000/ps-955c264e8f20fea3",
      permission_set_name = "Developers",
      principal_type = "GROUP",
      principal_name = "Developers"
    },
    {
      account = "222222

222

222",
      permission_set_arn = "arn:aws:sso:::permissionSet/ssoins-0000000000000000/ps-31d20e5987f0ce66",
      permission_set_name = "Developers",
      principal_type = "GROUP",
      principal_name = "Developers"
    },
  ]
}
```

### Example Assignments

- Users in the `Administrators` group should be assigned the `AdministratorAccess` permission set in the `production` account.
- Users in the `Developers` group should be assigned the `DeveloperAccess` permission set in the `production` account.
- Users in the `Developers` group should be assigned the `AdministratorAccess` permission set in the `sandbox` account.
