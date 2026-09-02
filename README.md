# GitHub-Action-AWS-s3
-----
## Project Overview

This project demonstrates a secure CI/CD deployment from GitHub Actions to Amazon S3 using GitHub OIDC federation.

Every push to the `main` branch triggers a GitHub Actions workflow that:

1. Checks out the repository
2. Obtains a short-lived OIDC token from GitHub
3. Uses AWS STS to assume a restricted IAM role
4. Synchronizes the repository contents to an S3 bucket
5. Publishes the application through S3 static website hosting

No long-lived AWS access keys are stored in GitHub.

--------
## Architecture Diagram

<img width="1536" height="1024" alt="ChatGPT Image Sep 2, 2026, 03_49_31 PM" src="https://github.com/user-attachments/assets/ad904ac1-5862-49ed-b9e3-f44c194b6a81" />

--------

## Setting up GitHub as an OIDC Identity Provider for AWS

1. In AWS got to IAM
2. Select "Identity providers"
3. Add provider
4. Choose "OpenID Connect"
5. Provider URL: https://token.actions.githubusercontent.com
6. Audience: sts.amazonaws.com
7. Add provider
8. Verify the provider. You should see something like this:

```
Provider
OpenID Connect

URL:
token.actions.githubusercontent.com

Audience:
sts.amazonaws.com

```

documentation on this can be found : https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws

--------
## Configuring the Role and Trust Policy
>>Create the IAM Role
1. IAM
2. Create roles
3. Under Trusted identity type, select "Web identity"
4. Identity provider: token.actions.githubusercontent.com
5. Audience: sts.amazonaws.com
6. For this project, we want AWS to allow only GitHub Actions originating from the specific repository to assume the role.
The URL for this repo is : https://github.com/Dekasrepo/github-actions-aws-s3
Therefore, for fields below, it would be filled as such:
GitHub organization : Dekasrepo.   (Dekasrepo is not a GitHub organization account, it is the GitHub owner/username)
GitHub repository: github-actions-aws-s3   (repository of our code)
GitHub branch: main  (we are pushing only to "main" branch)


>>Add Permissions
7. Create inline policy
8. For the bucket level: ListBucket
9. For Object level : s3:GetObject, s3:PutObject, s3:DeleteObject

``` JSON
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
```

10. Trust relationship should look something like this:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::<AWS_ACCOUNT_ID>:role/github-action-s3-object"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
                    "token.actions.githubusercontent.com:sub": "repo:Dekasrepo@181901279/github-actions-aws-s3@1354510926:ref:refs/heads/main"
                }
            }
        }
    ]
}

```
--------

## Push to GitHub and activate GitHub Actions
1. git init
2. git add .
3. git commit -m "my first commit" #choose your preferred commit message
4. git branch -M main
5. git remote add origin https://github.com/Dekasrepo/github-actions-aws-s3.git #for new repo
6. git push -u origin main


## Successful Deployment
<img width="658" height="491" alt="Screenshot 2026-09-02 at 13 52 11" src="https://github.com/user-attachments/assets/76784e4d-6267-4cab-bdbe-f373535ad460" />

------
## Troubleshooting

### OIDC AssumeRoleWithWebIdentity failure

The initial deployment failed with:

`Not authorized to perform sts:AssumeRoleWithWebIdentity`

The GitHub repository was created after July 15, 2026, so its OIDC
`sub` claim used the immutable repository/owner ID format.

The initial IAM trust policy used the legacy repository-name format,
which did not match the token's `sub` claim.

I updated the trust policy to match the immutable `sub` claim:

`repo:Dekasrepo@181901279/github-actions-aws-s3@1354510926:ref:refs/heads/main`

After updating the trust policy, the workflow successfully assumed the
IAM role and deployed the application to S3.

-------
## Security Considerations

- GitHub Actions authenticates to AWS using OIDC rather than long-lived
  AWS access keys.
- The IAM trust policy restricts role assumption to a specific repository
  and `main` branch.
- The IAM role uses only the S3 permissions required for deployment.
- The workflow explicitly grants `id-token: write` and `contents: read`.
- No AWS access keys or secret credentials are stored in the repository.
