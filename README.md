# AWS Identity and Access Management (IAM) Terraform module

## Features

1. **Cross-account access.** Define IAM roles using `iam_assumable_role` or `iam_assumable_roles` submodules in "resource AWS accounts (prod, staging, dev)" and IAM groups and users using `iam-group-with-assumable-roles-policy` submodule in "IAM AWS Account" to setup access controls between accounts. See [iam-group-with-assumable-roles-policy example](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/examples/iam-group-with-assumable-roles-policy) for more details.
1. **Individual IAM resources (users, roles, policies).** See usage snippets and [examples](https://github.com/terraform-aws-modules/terraform-aws-iam#examples) listed below.

## Usage

`iam-account`:

```hcl
module "iam_account" {
  source  = "github.com/jasonfhill/terraform-aws-iam/modules/iam-account"
  version = "~> 3.0"

  account_alias = "awesome-company"

  minimum_password_length = 37
  require_numbers         = false
}
```

`iam-assumable-role`:

```hcl
module "iam_assumable_role" {
  source  = "github.com/jasonfhill/terraform-aws-iam/modules/iam-assumable-role"
  version = "~> 3.0"

  trusted_role_arns = [
    "arn:aws:iam::307990089504:root",
    "arn:aws:iam::835367859851:user/anton",
  ]

  create_role = true

  role_name         = "custom"
  role_requires_mfa = true

  custom_role_policy_arns = [
    "arn:aws:iam::aws:policy/AmazonCognitoReadOnly",
    "arn:aws:iam::aws:policy/AlexaForBusinessFullAccess",
  ]
  number_of_custom_role_policy_arns = 2
}
```

`iam-assumable-role-with-oidc`:

```hcl
module "iam_assumable_role_with_oidc" {
  source  = "github.com/jasonfhill/terraform-aws-iam/modules/iam-assumable-role-with-oidc"
  version = "~> 3.0"

  create_role = true

  role_name = "role-with-oidc"

  tags = {
    Role = "role-with-oidc"
  }

  provider_url = "oidc.eks.eu-west-1.amazonaws.com/id/BA9E170D464AF7B92084EF72A69B9DC8"

  role_policy_arns = [
    "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy",
  ]
  number_of_role_policy_arns = 1
}
```

`iam-assumable-role-with-saml`:

```hcl
module "iam_assumable_role_with_saml" {
  source  = "github.com/jasonfhill/terraform-aws-iam/modules/iam-assumable-role-with-saml"
  version = "~> 3.0"

  create_role = true

  role_name = "role-with-saml"

  tags = {
    Role = "role-with-saml"
  }

  provider_id = "arn:aws:iam::235367859851:saml-provider/idp_saml"

  role_policy_arns = [
    "arn:aws:iam::aws:policy/ReadOnlyAccess"
  ]
  number_of_role_policy_arns = 1
}
```

`iam-assumable-roles`:

```hcl
module "iam_assumable_roles" {
  source  = "github.com/jasonfhill/terraform-aws-iam/modules/iam-assumable-roles"
  version = "~> 3.0"

  trusted_role_arns = [
    "arn:aws:iam::307990089504:root",
    "arn:aws:iam::835367859851:user/anton",
  ]

  create_admin_role = true

  create_poweruser_role = true
  poweruser_role_name   = "developer"

  create_readonly_role       = true
  readonly_role_requires_mfa = false
}
```

`iam-assumable-roles-with-saml`:

```hcl
module "iam_assumable_roles_with_saml" {
  source  = "github.com/jasonfhill/terraform-aws-iam/modules/iam-assumable-roles-with-saml"
  version = "~> 3.0"

  create_admin_role = true

  create_poweruser_role = true
  poweruser_role_name   = "developer"

  create_readonly_role = true

  provider_id   = "arn:aws:iam::235367859851:saml-provider/idp_saml"
}
```

`iam-user`:

```hcl
module "iam_user" {
  source  = "github.com/jasonfhill/terraform-aws-iam/modules/iam-user"
  version = "~> 3.0"

  name          = "vasya.pupkin"
  force_destroy = true

  pgp_key = "keybase:test"

  password_reset_required = false
}
```

`iam-policy`:

```hcl
module "iam_policy" {
  source  = "github.com/jasonfhill/terraform-aws-iam/modules/iam-policy"
  version = "~> 3.0"

  name        = "example"
  path        = "/"
  description = "My example policy"

  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "ec2:Describe*"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
EOF
}
```

`iam-group-with-assumable-roles-policy`:

```hcl
module "iam_group_with_assumable_roles_policy" {
  source  = "github.com/jasonfhill/terraform-aws-iam/modules/iam-group-with-assumable-roles-policy"
  version = "~> 3.0"

  name = "production-readonly"

  assumable_roles = [
    "arn:aws:iam::835367859855:role/readonly"  # these roles can be created using `iam_assumable_roles` submodule
  ]

  group_users = [
    "user1",
    "user2"
  ]
}
```

`iam-group-with-policies`:

```hcl
module "iam_group_with_policies" {
  source  = "github.com/jasonfhill/terraform-aws-iam/modules/iam-group-with-policies"
  version = "~> 3.0"

  name = "superadmins"

  group_users = [
    "user1",
    "user2"
  ]

  attach_iam_self_management_policy = true

  custom_group_policy_arns = [
    "arn:aws:iam::aws:policy/AdministratorAccess",
  ]

  custom_group_policies = [
    {
      name   = "AllowS3Listing"
      policy = data.aws_iam_policy_document.sample.json
    }
  ]
}
```

## IAM Best Practices

AWS published [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### 1. Create Individual IAM Users

Use [iam-user module](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/modules/iam-user) module to manage IAM users.

### 2. Use AWS Defined Policies to Assign Permissions Whenever Possible

Use [iam-assumable-roles module](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/modules/iam-assumable-roles) to create IAM roles with managed policies to support common tasks (admin, poweruser or readonly).

### 3. Use Groups to Assign Permissions to IAM Users

Use [iam-group-with-assumable-roles-policy module](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/modules/iam-group-with-assumable-roles-policy) to manage IAM groups of users who can assume roles.

Use [iam-group-with-policies module](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/modules/iam-group-with-policies) to manage IAM groups of users where specified IAM policies are allowed.

### 4. Configure a Strong Password Policy for Your Users

Use [iam-account module](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/modules/iam-account) to set password policy for your IAM users.

### 5. Enable MFA for Privileged Users

Terraform can't configure MFA for the user. It is only possible via [AWS Console](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html) and [AWS CLI](https://docs.aws.amazon.com/cli/latest/reference/iam/enable-mfa-device.html).

### 6. Delegate by Using Roles Instead of by Sharing Credentials

[iam-assumable-role](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/modules/iam-assumable-role), [iam-assumable-roles](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/modules/iam-assumable-roles), [iam-assumable-roles-with-saml](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/modules/iam-assumable-roles-with-saml) and [iam-group-with-assumable-roles-policy](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/modules/iam-group-with-assumable-roles-policy) modules provide complete set of functionality required for this.

### 7. Use Policy Conditions for Extra Security

[iam-assumable-roles module](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/modules/iam-assumable-roles) can be configured to require valid MFA token when different roles are assumed (for example, admin role requires MFA, but readonly - does not).

### 8. Create IAM Policies

Use [iam-policy module](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/modules/iam-policy) module to manage IAM policy.

## Examples

- [iam-account](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/examples/iam-account) - Set AWS account alias and password policy
- [iam-assumable-role](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/examples/iam-assumable-role) - Create individual IAM role which can be assumed from specified ARNs (AWS accounts, IAM users, etc)
- [iam-assumable-role-with-oidc](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/examples/iam-assumable-role-with-oidc) - Create individual IAM role which can be assumed from specified subjects federated with a OIDC Identity Provider
- [iam-assumable-role-with-saml](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/examples/iam-assumable-role-with-saml) - Create individual IAM role which can be assumed by users with a SAML Identity Provider
- [iam-assumable-roles](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/examples/iam-assumable-roles) - Create several IAM roles which can be assumed from specified ARNs (AWS accounts, IAM users, etc)
- [iam-assumable-roles-with-saml](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/examples/iam-assumable-roles-with-saml) - Create several IAM roles which can be assumed by users with a SAML Identity Provider
- [iam-group-with-assumable-roles-policy](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/examples/iam-group-with-assumable-roles-policy) - IAM group with users who are allowed to assume IAM roles in the same or in separate AWS account
- [iam-group-with-policies](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/examples/iam-group-with-policies) - IAM group with users who are allowed specified IAM policies (eg, "manage their own IAM user")
- [iam-group-complete](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/examples/iam-group-complete) - IAM group with users who are allowed to assume IAM roles in another AWS account and have access to specified IAM policies
- [iam-user](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/examples/iam-user) - Add IAM user, login profile and access keys (with PGP enabled or disabled)
- [iam-policy](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/examples/iam-policy) - Create IAM policy

## Authors

Module is maintained by [Anton Babenko](https://github.com/antonbabenko) with help from [these awesome contributors](https://github.com/terraform-aws-modules/terraform-aws-iam/graphs/contributors).

## License

Apache 2 Licensed. See [LICENSE](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/LICENSE) for full details.
