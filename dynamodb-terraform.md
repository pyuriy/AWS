## Create a DynamoDB instance on AWS with terraform and connect to it via NoSQL Workbench app

### Prerequisites
Before proceeding, ensure you have:
- An AWS account with appropriate permissions (e.g., `AmazonDynamoDBFullAccess` policy).
- Terraform installed (version 1.0+). Download from [terraform.io](https://www.terraform.io/downloads).
- AWS CLI installed and configured with credentials (`aws configure`).
- NoSQL Workbench downloaded from [AWS NoSQL Workbench](https://docs.aws.amazon.com/dynamodb/latest/developerguide/workbench.html) (available for Windows, macOS, Linux).

### Step 1: Set Up Terraform Configuration
Create a new directory for your project (e.g., `dynamodb-terraform`), and inside it, create a file named `main.tf` with the following Terraform code. This creates a simple DynamoDB table named `MyTable` with:
- Pay-per-request billing.
- A single partition key (`id` as a string).
- TTL enabled on an attribute named `expiration` (optional, for data expiration).

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Configure the AWS Provider
provider "aws" {
  region = "us-east-1"  # Change to your preferred region
}

# Create a DynamoDB table
resource "aws_dynamodb_table" "my_table" {
  name           = "MyTable"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "id"

  attribute {
    name = "id"
    type = "S"  # String type for partition key
  }

  # Optional: Enable TTL for automatic data expiration
  ttl {
    attribute_name = "expiration"
    enabled        = true
  }

  tags = {
    Name = "MyTerraformTable"
  }
}

# Output the table name and ARN for reference
output "table_name" {
  value = aws_dynamodb_table.my_table.name
}

output "table_arn" {
  value = aws_dynamodb_table.my_table.arn
}
```

### Step 2: Initialize and Apply Terraform
Run these commands in your project directory:
1. Initialize Terraform to download providers:
   ```
   terraform init
   ```
2. Review the planned changes (optional):
   ```
   terraform plan
   ```
3. Create the DynamoDB table:
   ```
   terraform apply
   ```
   - Type `yes` when prompted.
   - This will output the table name and ARN (e.g., `MyTable` and `arn:aws:dynamodb:us-east-1:123456789012:table/MyTable`).

4. Verify in AWS Console: Log in to [AWS DynamoDB Console](https://console.aws.amazon.com/dynamodb), select your region, and confirm the table exists.

### Step 3: Connect to DynamoDB via NoSQL Workbench
1. Launch NoSQL Workbench and select **Operation Builder** (or **Data Modeler** for schema design).
2. Click **Connect to DynamoDB** > **AWS Profile**.
3. Choose your AWS profile (from `aws configure`) or enter credentials manually:
   - Access Key ID
   - Secret Access Key
   - Default region (match your Terraform region, e.g., `us-east-1`).
4. If using an IAM role or endpoint, select **Connection String** instead and use format: `dynamodb.us-east-1.amazonaws.com` (adjust region).
5. Test the connection. If successful, you'll see your tables (including `MyTable`).
6. Explore: Add sample data via **Add** operation (e.g., insert items with `id: "test-1"`), query, or scan the table.

### Cleanup (Optional)
To destroy the resources:
```
terraform destroy
```
Type `yes` when prompted.

### Troubleshooting
- **Permission errors**: Ensure your AWS credentials have DynamoDB permissions.
- **Region mismatch**: Tables are region-specific; ensure NoSQL Workbench uses the same region as Terraform.
- **Terraform errors**: Run `terraform validate` to check syntax.
- For advanced features (e.g., global secondary indexes), extend the `aws_dynamodb_table` resource accordingly. Refer to [Terraform AWS DynamoDB docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/dynamodb_table).


## Steps to configure the required access on AWS side
### Prerequisites for AWS Access Configuration
Before configuring access, ensure:
- You have an AWS account and are signed in to the [AWS Management Console](https://console.aws.amazon.com/) as an IAM administrator (or have permissions to manage IAM).
- Decide on the authentication method: IAM user (recommended for long-term access) or IAM role (for EC2 or temporary sessions). This guide focuses on an IAM user for simplicity.

The required permissions for creating a DynamoDB table via Terraform and connecting/querying via NoSQL Workbench include:
- `dynamodb:CreateTable`, `dynamodb:DescribeTable`, `dynamodb:PutItem`, `dynamodb:GetItem`, `dynamodb:UpdateItem`, `dynamodb:DeleteItem`, `dynamodb:Scan`, `dynamodb:Query` (and related actions).
- Use the managed policy `AmazonDynamoDBFullAccess` for full access, or a custom policy for least privilege.

### Step 1: Create an IAM User (If You Don't Have One)
1. In the AWS Console, navigate to **IAM** > **Users** > **Create user**.
2. Enter a user name (e.g., `dynamodb-terraform-user`).
3. Select **Provide user access to the AWS Management Console** if you want console login (optional; skip for programmatic access only).
4. On the **Set permissions** page, choose **Attach policies directly**.
5. Search for and select `AmazonDynamoDBFullAccess`.
6. Click **Next** > **Create user**.
7. Note the user ARN (e.g., `arn:aws:iam::123456789012:user/dynamodb-terraform-user`).

### Step 2: Generate Access Keys for the IAM User
1. In IAM > **Users**, select your new user > **Security credentials** tab.
2. Under **Access keys**, click **Create access key**.
3. Select **Command Line Interface (CLI)** or **Application running outside AWS** as the use case.
4. Acknowledge the recommendations, then click **Create access key**.
5. Download the `.csv` file containing:
   - **Access key ID** (e.g., `AKIAIOSFODNN7EXAMPLE`).
   - **Secret access key** (e.g., `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`).
6. **Important**: Store these securely; you won't see the secret key again. Delete unused keys to avoid security risks.

### Step 3: Configure AWS CLI with the Credentials
1. Install AWS CLI if not already (download from [AWS CLI docs](https://aws.amazon.com/cli/)).
2. Run `aws configure` in your terminal.
3. Enter:
   - **AWS Access Key ID**: From Step 2.
   - **AWS Secret Access Key**: From Step 2.
   - **Default region name**: e.g., `us-east-1` (match your Terraform config).
   - **Default output format**: `json` (optional).
4. Verify: Run `aws sts get-caller-identity` to confirm the user is active.

This creates a profile (default) that Terraform and NoSQL Workbench can use.

### Step 4: (Optional) Create a Custom IAM Policy for Least Privilege
If you prefer not to use the full `AmazonDynamoDBFullAccess` policy:
1. In IAM > **Policies** > **Create policy**.
2. Switch to **JSON** tab and paste:
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "dynamodb:CreateTable",
                   "dynamodb:DescribeTable",
                   "dynamodb:PutItem",
                   "dynamodb:GetItem",
                   "dynamodb:UpdateItem",
                   "dynamodb:DeleteItem",
                   "dynamodb:Scan",
                   "dynamodb:Query",
                   "dynamodb:BatchGetItem",
                   "dynamodb:BatchWriteItem",
                   "dynamodb:UpdateTimeToLive"
               ],
               "Resource": "arn:aws:dynamodb:*:*:table/MyTable"  // Replace with your table ARN or use "*" for all tables
           }
       ]
   }
   ```
3. Click **Next** > Add tags (optional) > Name it (e.g., `DynamoDBTerraformAccess`) > **Create policy**.
4. Attach this policy to your IAM user (IAM > Users > [User] > Add permissions > Attach policies).

### Step 5: Test Access
1. **For Terraform**: Run `terraform plan` in your project directory. It should detect no permission issues.
2. **For NoSQL Workbench**:
   - Launch the app > **Connect to DynamoDB** > **AWS Profile**.
   - Select your profile (from `aws configure`).
   - If connected, list tables to see `MyTable` (after `terraform apply`).
3. In AWS Console: IAM > Users > [User] > **Access Advisor** to monitor usage.

### Additional Notes
- **Security Best Practices**: Use MFA on the IAM user, rotate keys regularly, and avoid root account credentials.
- **For Roles (e.g., EC2)**: Instead of a user, create an IAM role with the policy attached, and assume it via `aws sts assume-role`.
- **Troubleshooting**:
  - **Access Denied**: Check policy attachments and resource ARNs (e.g., region-specific).
  - **No Tables Visible**: Ensure the region matches and table is created.
- Refer to [AWS IAM Docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/) for advanced setups.

This setup enables seamless access for both tools. If you need steps for a specific scenario (e.g., cross-account access), provide more details!
