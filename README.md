# Terraform-S3-IAM-user-S3-policy
Using Terraform to create IAM user/S3 bucket and attaching read only policy to IAM user using variables
CODE:

#Provider profile and region in which all the resources will create
provider "aws" {
  
  region  = var.region
}
variable "region" {
  type = string
}
variable "aws_s3_bucket" {
  description = "bucketname"
  type = string
  
}
variable "newuser" {
  description = "enter the new user"
  type = string
  
}
resource "aws_iam_user" "iamuser" {
  name = var.newuser
  
}
resource "aws_iam_access_key" "new_user_access_key" {
  user = aws_iam_user.iamuser.name  # Generate access keys for the new IAM user
}

output "iam_user_access_key" {
  value     = aws_iam_access_key.new_user_access_key.id
  sensitive = true # Mark the access key as sensitive to prevent it from being displayed in logs or outputs
}

output "iam_user_secret_key" {
  value     = aws_iam_access_key.new_user_access_key.secret
  sensitive = true # Mark the secret key as sensitive for the same reason
}
data "aws_iam_policy_document" "s3_access_policy" {
  statement {
    actions   = ["s3:GetObject"]          # Allow the user to read objects from S3
    resources = [var.aws_s3_bucket]       # Specify which S3 bucket they can access
  }
}
resource "aws_iam_policy" "s3_access_policy" {
  name        = "s3-access-policy-for-${var.newuser}"
  description = "Policy allowing read access to S3"
  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Effect   = "Allow",
        Action   = "s3:GetObject",
        Resource = "arn:aws:s3:::example-bucket/*"
      }
    ]
  })
}
resource "aws_iam_policy_attachment" "attach_s3_policy" {
  name       = "attach-s3-access-policy"
  users      = [aws_iam_user.iamuser.name]   # Attach the policy to the newly created IAM user
  policy_arn = aws_iam_policy.s3_access_policy.arn

  depends_on = [aws_iam_user.iamuser]        # Ensure user is created before attaching policy
}
#Resource to create s3 bucket
resource "aws_s3_bucket" "demo-bucket"{
  bucket = var.aws_s3_bucket
  tags = {
    Name = "S3Bucket"
  }
}
output "bucketnameis" {
  value = aws_s3_bucket.demo-bucket.id
}
output "bucketarn" {
  value = aws_s3_bucket.demo-bucket.arn
  
}

