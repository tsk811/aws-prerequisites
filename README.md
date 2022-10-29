# Prerequisites

The below setup needs to be completed manually before using the other projects. All of these are mandatory for Cross Account Deployments.

### Non Prod account
-----

This account will host the pipeline for Infra and App deployments.

#### Resources 


* Create an **Artifact Bucket** to store the artifacts generated from the pipeline.
    1. Add below policy to the bucket.
        > {
                "Version": "2012-10-17",
                "Id": "Policy1553183091390",
                "Statement": [
                    {
                        "Sid": "PermissionForProdAccount",
                        "Effect": "Allow",
                        "Principal": {
                            "AWS": "arn:aws:iam::<*PROD_ACCOUNT_ID*>:root"
                        },
                        "Action": [
                            "s3:Get*",
                            "s3:ListBucket"
                        ],
                        "Resource": [
                            "arn:aws:s3:::<*ARTIFACT_BUCKET_NAME*>/*",
                            "arn:aws:s3:::<*ARTIFACT_BUCKET_NAME*>"
                        ]
                    }
                ]
            }

Creating the above bucket(without policy) is mandatory even if you are not doing Cross Account Deployments.

* Create a **KMS Key**.
    1. Add below policy
        > {
                "Version": "2012-10-17",
                "Id": "KeyPolicy",
                "Statement": [
                    {
                        "Sid": "Allow use of the key",
                        "Effect": "Allow",
                        "Principal": {
                            "AWS": "arn:aws:iam::<*DEV_ACCOUNT_ID*>:root"
                        },
                        "Action": "kms:*",
                        "Resource": "arn:aws:kms:us-east-1:<*DEV_ACCOUNT_ID*>:key/<*KMS_KEY_ID*>"
                    },
                    {
                        "Sid": "Allow use of the key",
                        "Effect": "Allow",
                        "Principal": {
                            "AWS": "arn:aws:iam::<*PROD_ACCOUNT_ID*>:root"
                        },
                        "Action": [
                            "kms:DescribeKey",
                            "kms:GenerateDataKey*",
                            "kms:Encrypt",
                            "kms:ReEncrypt*",
                            "kms:Decrypt"
                        ],
                        "Resource": "arn:aws:kms:us-east-1:<*DEV_ACCOUNT_ID*>:key/<*KMS_KEY_ID*>"
                    }
                ]
            }

* Export the configs in the Parameter store.
    1. /configs/CrossAccountRole --> ARN of the **Cross Account Role** in Production account.
    2. /configs/ProdDeploymentRole --> ARN of the **Deployment Role** in Production account.
    3. /configs/artifactBucket --> Name of the **Artifact Bucket**.
    4. /configs/kms_key --> ARN of the **KMS Key**.


* Create the below secret in Secrets Manager.
    1. /secrets/github_token --> {"token": "<*Github_OAuth_token*>"} --> This needed to checkout the source code.


### Production account
-----

#### Resources
* Create a **Cross Account Role** with the below policies.
    1. Cross Account policy
        > {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Action": [
                            "cloudformation:*",
                            "iam:PassRole"
                        ],
                        "Resource": "*"
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "s3:Get*",
                            "s3:ListBucket"
                        ],
                        "Resource": [
                            "arn:aws:s3:::<*ARTIFACT_BUCKET_NAME*>/*",
                            "arn:aws:s3:::<*ARTIFACT_BUCKET_NAME*>"
                        ]
                    }
                ]
                } 
    2. Cross Account KMS Policy
        > {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": [
                        "kms:DescribeKey",
                        "kms:GenerateDataKey*",
                        "kms:Encrypt",
                        "kms:ReEncrypt*",
                        "kms:Decrypt"
                    ],
                    "Resource": [
                        "arn:aws:kms:us-east-1:<*DEV_ACCOUNT_ID*>:key/<*KMS_KEY_ID*>"
                    ]
                }
            ]
        }
* Create a CloudFormation **Deployment Role** to allow CloudFormation to create resources.
    1. Policy with Admin privileges or restricted permissions based on the resources which you need.
