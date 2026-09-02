# github-actions-aws-s3

This readme explains the process used to establish a trust relationship between GitHub, GitHub Actions and AWS. 
It also shows the steps to set up the IAM role for the identity provider to be able to execute actions on AWS resosurces.

## To setup GitHub As Identity Provider for to AWS

1. In AWS got to IAM
2. Select "Identity providers"
3. Add provider
4. Choose "OpeinID Connect"
5. Provicer URL: https://token.actions.githubusercontent.com
6. Audience: sts.amazonaws.com
7. Add provider
8. Verify the provider. You should see something like this:

'''
Provider
OpenID Connect

URL:
token.actions.githubusercontent.com

Audience:
sts.amazonaws.com

'''

documentation on this can be found : https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws


## Configuring the Role and Trust Policy
>>Create the IAM Role
1. IAM
2. Create roles
3. Under Trusted identity type, select "Web identity"
4. Identity provider: token.actions.githubusercontent.com
5. Audience: sts.amazonaws.com
6. For this project, we want AWS to allow only GitHub Actions orginiating from the specific repository to assume the role.
The URL for this repo is : https://github.com/Dekasrepo/github-actions-aws-s3
Therefore, for fields below, it would be filled as such:
GitHub organization : Dekasrepo.   (Dekasrepo is not a GitHub organization account, it is the GitHub owner/username)
GitHub repository: github-actions-aws-s3   (repostory of our code)
GitHub branch: main  (we are pushing only to "main" branch)


>>Add Permissions
7. Create inine policy
8. For the bukcet level: ListBucket
9. For Object level : s3:GetObject, s3:PutObject, s3:DeleteObject
''' JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
''''

