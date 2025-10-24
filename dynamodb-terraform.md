## Steps for and terraform code to create a DynamoDB instance on AWS and connect to it via NoSQL Workbench app

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
