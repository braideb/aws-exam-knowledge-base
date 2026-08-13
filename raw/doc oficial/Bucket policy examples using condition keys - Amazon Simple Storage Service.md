---
title: "Bucket policy examples using condition keys - Amazon Simple Storage Service"
source: "https://docs.aws.amazon.com/AmazonS3/latest/userguide/amazon-s3-policy-keys.html"
author:
published:
created: 2026-07-19
description: "You can use access policy language to specify conditions when you grant permissions. You can use the optional Condition element, or Condition block, to specify conditions for when a policy is in effect."
tags:
  - "clippings"
---
Bucket policy examples using condition keys - Amazon Simple Storage Service

You can use access policy language to specify conditions when you grant permissions. You can use the optional `Condition` element, or `Condition` block, to specify conditions for when a policy is in effect.

For policies that use Amazon S3 condition keys for object and bucket operations, see the following examples. For more information about condition keys, see [Policy condition keys for Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security_iam_service-with-iam.html#security_iam_service-with-iam-id-based-policies-conditionkeys). For a complete list of Amazon S3 actions, condition keys, and resources that you can specify in policies, see [Actions, resources, and condition keys for Amazon S3](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazons3.html) in the *Service Authorization Reference*.

For more information about the permissions to S3 API operations by S3 resource types, see [Required permissions for Amazon S3 API operations](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-with-s3-policy-actions.html).

## Examples: Amazon S3 condition keys for object operations

The following examples show how you can use Amazon S3‐specific condition keys for object operations. For a complete list of Amazon S3 actions, condition keys, and resources that you can specify in policies, see [Actions, resources, and condition keys for Amazon S3](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazons3.html) in the *Service Authorization Reference*.

For more information about the permissions to S3 API operations by S3 resource types, see [Required permissions for Amazon S3 API operations](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-with-s3-policy-actions.html).

Several of the example policies show how you can use conditions keys with [PUT Object](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectPUT.html) operations. PUT Object operations allow access control list (ACL)–specific headers that you can use to grant ACL-based permissions. By using these condition keys, you can set a condition to require specific access permissions when the user uploads an object. You can also grant ACL–based permissions with the PutObjectAcl operation. For more information, see [PutObjectAcl](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObjectAcl.html) in the *Amazon S3 Amazon Simple Storage Service API Reference*. For more information about ACLs, see [Access control list (ACL) overview](https://docs.aws.amazon.com/AmazonS3/latest/userguide/acl-overview.html).

### Example 1: Granting s3:PutObject permission requiring that objects be stored using server-side encryption

Suppose that Account A owns a bucket. The account administrator wants to grant Jane, a user in Account A, permission to upload objects with the condition that Jane always request server-side encryption with Amazon S3 managed keys (SSE-S3). The Account A administrator can specify this requirement by using the `s3:x-amz-server-side-encryption` condition key as shown. The key-value pair in the following `Condition` block specifies the `s3:x-amz-server-side-encryption` condition key and SSE-S3 (`AES256`) as the encryption type:

```bash
"Condition": {
     "StringNotEquals": {
         "s3:x-amz-server-side-encryption": "AES256"
     }}
```

When testing this permission by using the AWS CLI, you must add the required encryption by using the `--server-side-encryption` parameter, as shown in the following example. To use this example command, replace the `` `user input 						placeholders` `` with your own information.

```
aws s3api put-object --bucket amzn-s3-demo-bucket --key HappyFace.jpg --body c:\HappyFace.jpg --server-side-encryption "AES256" --profile AccountAadmin
```

### Example 2: Granting s3:PutObject permission to copy objects with a restriction on the copy source

In a `PUT` object request, when you specify a source object, the request is a copy operation (see [CopyObject](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectCOPY.html)). Accordingly, the bucket owner can grant a user permission to copy objects with restrictions on the source, for example:

- Allow copying objects only from the specified source bucket (for example, `` `amzn-s3-demo-source-bucket` ``).
- Allow copying objects from the specified source bucket and only the objects whose key name prefix starts with as specific prefix, such as `` `public/` `` (for example, ``` `amzn-s3-demo-source-bucket``/public/*` ```).
- Allow copying only a specific object from the source bucket (for example, ``` `amzn-s3-demo-source-bucket``/example.jpg` ```).

The following bucket policy grants a user (`` `Dave` ``) the `s3:PutObject` permission. This policy allows him to copy objects only with a condition that the request include the `s3:x-amz-copy-source` header and that the header value specify the ``` /`amzn-s3-demo-source-bucket``/public/*` ``` key name prefix. To use this example policy, replace the `` `user 						input placeholders` `` with your own information.

anchor

```json
{
    "Version":"2012-10-17",
    "Statement": [
       {
            "Sid": "cross-account permission to user in your own account",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:user/Dave"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::amzn-s3-demo-source-bucket/*"
        },
        {
            "Sid": "Deny your user permission to upload object if copy source is not /bucket/prefix",
            "Effect": "Deny",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:user/Dave"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::amzn-s3-demo-source-bucket/*",
            "Condition": {
                "StringNotLike": {
                    "s3:x-amz-copy-source": "amzn-s3-demo-source-bucket/public/*"
                }
            }
        }
    ]
}
```

###### Test the policy with the AWS CLI

You can test the permission using the AWS CLI `copy-object` command. You specify the source by adding the `--copy-source` parameter; the key name prefix must match the prefix allowed in the policy. You need to provide the user Dave credentials using the `--profile` parameter. For more information about setting up the AWS CLI, see [Developing with Amazon S3 using the AWS CLI](https://docs.aws.amazon.com/AmazonS3/latest/API/setup-aws-cli.html) in the *Amazon S3 API Reference*.

```
aws s3api copy-object --bucket amzn-s3-demo-source-bucket --key HappyFace.jpg 
--copy-source amzn-s3-demo-source-bucket/public/PublicHappyFace1.jpg --profile AccountADave
```

###### Give permission to copy only a specific object

The preceding policy uses the `StringNotLike` condition. To grant permission to copy only a specific object, you must change the condition from `StringNotLike` to `StringNotEquals` and then specify the exact object key, as shown in the following example. To use this example command, replace the `` `user input 						placeholders` `` with your own information.

```
"Condition": {
       "StringNotEquals": {
           "s3:x-amz-copy-source": "amzn-s3-demo-source-bucket/public/PublicHappyFace1.jpg"
       }
}
```

### Example 3: Granting access to a specific version of an object

Suppose that Account A owns a versioning-enabled bucket. The bucket has several versions of the `` `HappyFace.jpg` `` object. The Account A administrator now wants to grant the user `` `Dave` `` permission to get only a specific version of the object. The account administrator can accomplish this by granting the user `` `Dave` `` the `s3:GetObjectVersion` permission conditionally, as shown in the following example. The key-value pair in the `Condition` block specifies the `s3:VersionId` condition key. In this case, to retrieve the object from the specified versioning-enabled bucket, `` `Dave` `` needs to know the exact object version ID. To use this example policy, replace the `` `user input 						placeholders` `` with your own information.

For more information, see [GetObject](https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetObject.html) in the *Amazon Simple Storage Service API Reference*.

anchor

```json
{
    "Version":"2012-10-17",
    "Statement": [
        {
            "Sid": "statement1",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:user/Dave"
            },
            "Action": "s3:GetObjectVersion",
            "Resource": "arn:aws:s3:::amzn-s3-demo-bucket/HappyFace.jpg"
        },
        {
            "Sid": "statement2",
            "Effect": "Deny",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:user/Dave"
            },
            "Action": "s3:GetObjectVersion",
            "Resource": "arn:aws:s3:::amzn-s3-demo-bucket/HappyFace.jpg",
            "Condition": {
                "StringNotEquals": {
                    "s3:VersionId": "AaaHbAQitwiL_h47_44lRO2DDfLlBO5e"
                }
            }
        }
    ]
}
```

###### Test the policy with the AWS CLI

You can test the permissions in this policy by using the AWS CLI `get-object` command with the `--version-id` parameter to identify the specific object version to retrieve. The command retrieves the specified version of the object and saves it to the `` `OutputFile.jpg` `` file.

```
aws s3api get-object --bucket amzn-s3-demo-bucket --key HappyFace.jpg OutputFile.jpg --version-id AaaHbAQitwiL_h47_44lRO2DDfLlBO5e --profile AccountADave
```

### Example 4: Granting permissions based on object tags

For examples of how to use object tagging condition keys with Amazon S3 operations, see [Tagging and access control policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/tagging-and-policies.html).

### Example 5: Restricting access by the AWS account ID of the bucket owner

You can use either the `aws:ResourceAccount` or `s3:ResourceAccount` condition key to write IAM or virtual private cloud (VPC) endpoint policies that restrict user, role, or application access to the Amazon S3 buckets that are owned by a specific AWS account ID. You can use these condition keys to restrict clients within your VPC from accessing buckets that you don't own.

However, be aware that some AWS services rely on access to AWS managed buckets. Therefore, using the `aws:ResourceAccount` or `s3:ResourceAccount` key in your IAM policy might also affect access to these resources. For more information, see the following resources:

- [Restrict access to buckets in a specified AWS account](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html#bucket-policies-s3) in the *AWS PrivateLink Guide*
- [Restrict access to buckets that Amazon ECR uses](https://docs.aws.amazon.com/AmazonECR/latest/userguide/vpc-endpoints.html#ecr-minimum-s3-perms) in the *Amazon ECR Guide*
- [Provide required access to Systems Manager for AWS managed Amazon S3 buckets](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent-minimum-s3-permissions.html) in the *AWS Systems Manager Guide*

For more information about the `aws:ResourceAccount` and `s3:ResourceAccount` condition keys and examples that show how to use them, see [Limit access to Amazon S3 buckets owned by specific AWS accounts](https://aws.amazon.com/blogs/storage/limit-access-to-amazon-s3-buckets-owned-by-specific-aws-accounts/) in the *AWS Storage Blog*.

### Example 6: Requiring a minimum TLS version

You can use the `s3:TlsVersion` condition key to write IAM, virtual private cloud endpoint (VPCE), or bucket policies that restrict user or application access to Amazon S3 buckets based on the TLS version that's used by the client. You can use this condition key to write policies that require a minimum TLS version.

###### Note

When AWS services make calls to other AWS services on your behalf (service-to-service calls), certain network-specific authorization context is redacted, including `s3:TlsVersion`, `aws:SecureTransport`, `aws:SourceIp`, and `aws:VpcSourceIp`. If your policy uses these condition keys with `Deny` statements, AWS service principals might be unintentionally blocked. To allow AWS services to work properly while maintaining your security requirements, exclude service principals from your `Deny` statements by adding the `aws:PrincipalIsAWSService` condition key with a value of `false`. For example:

```json
{
  "Effect": "Deny",
  "Action": "s3:*",
  "Resource": "*",
  "Condition": {
    "Bool": {
      "aws:SecureTransport": "false",
      "aws:PrincipalIsAWSService": "false"
    }
  }
}
```

This policy denies access to S3 operations when HTTPS is not used (`aws:SecureTransport` is false), but only for non-AWS service principals. This ensures your conditional restrictions apply to all principals except AWS service principals.

###### Example

The following example bucket policy *denies* `PutObject` requests by clients that have a TLS version earlier than 1.2, for example, 1.1 or 1.0. To use this example policy, replace the `` `user input placeholders` `` with your own information.

anchor

```json
{
    "Version":"2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": [
                "arn:aws:s3:::amzn-s3-demo-bucket1",
                "arn:aws:s3:::amzn-s3-demo-bucket1/*"
            ],
            "Condition": {
                "NumericLessThan": {
                    "s3:TlsVersion": 1.2
                }
            }
        }
    ]
}
```

###### Example

The following example bucket policy *allows* `PutObject` requests by clients that have a TLS version later than 1.1, for example, 1.2, 1.3, or later:

anchor

```json
{
    "Version":"2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": [
                "arn:aws:s3:::amzn-s3-demo-bucket1",
                "arn:aws:s3:::amzn-s3-demo-bucket1/*"
            ],
            "Condition": {
                "NumericGreaterThan": {
                    "s3:TlsVersion": 1.1
                }
            }
        }
    ]
}
```

### Example 7: Excluding certain principals from a Deny statement

The following bucket policy denies `s3:GetObject` access to the `` `amzn-s3-demo-bucket` ``, except to principals with the account number `` `123456789012` ``. To use this example policy, replace the `` `user input 					placeholders` `` with your own information.

anchor

```json
{
  "Version":"2012-10-17",
  "Statement": [
    {
      "Sid": "DenyAccessFromPrincipalNotInSpecificAccount",
      "Principal": {
        "AWS": "*"
      },
      "Action": "s3:GetObject",
      "Effect": "Deny",
      "Resource": [
        "arn:aws:s3:::amzn-s3-demo-bucket/*"
      ],
      "Condition": {
        "StringNotEquals": {
          "aws:PrincipalAccount": [
            "123456789012"
          ]
        }
      }
    }
  ]
}
```

### Example 8: Enforcing clients to conditionally upload objects based on object key names or ETags

With conditional writes, you can add an additional header to your `WRITE` requests in order to specify preconditions for your S3 operation. This header specifies a condition that, if not met, will result in the S3 operation failing. For example you can prevent overwrites of existing data by validating there is no object with the same key name already in your bucket during object upload. You can alternatively check an object's entity tag (ETag) in Amazon S3 before writing an object.

For bucket policy examples that use conditions in a bucket policy to enforce conditional writes, see [Enforce conditional writes on Amazon S3 buckets](https://docs.aws.amazon.com/AmazonS3/latest/userguide/conditional-writes-enforce.html).

### Example 9: Restricting annotation access by name prefix

The following bucket policy grants the `s3user` user permission to create, retrieve, and delete annotations that have names beginning with `ml-output.`. This allows you to restrict annotation access by annotation name prefix.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:user/s3user"
            },
            "Action": [
                "s3:PutObjectAnnotation",
                "s3:GetObjectAnnotation",
                "s3:DeleteObjectAnnotation"
            ],
            "Resource": "arn:aws:s3:::amzn-s3-demo-bucket/*",
            "Condition": {
                "StringLike": {
                    "s3:annotation-prefix": "ml-output.*"
                }
            }
        }
    ]
}
```

### Example 10: Requiring conditional writes for annotation operations

The following bucket policy denies `PutObjectAnnotation` and `DeleteObjectAnnotation` requests unless the caller includes the `x-amz-object-if-match` conditional header. This enforces optimistic concurrency control for all annotation writes on the bucket.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "RequireConditionalWritesForAnnotations",
            "Effect": "Deny",
            "Principal": "*",
            "Action": [
                "s3:PutObjectAnnotation",
                "s3:DeleteObjectAnnotation"
            ],
            "Resource": "arn:aws:s3:::amzn-s3-demo-bucket/*",
            "Condition": {
                "Null": {
                    "s3:x-amz-object-if-match": "true"
                }
            }
        }
    ]
}
```

### Example 11: Limiting pagination size for ListObjectAnnotations

The following bucket policy limits the `max-annotation-results` parameter to 100 or fewer for all `ListObjectAnnotations` requests. This prevents callers from requesting excessively large result sets.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "LimitAnnotationListPageSize",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:ListObjectAnnotation",
            "Resource": "arn:aws:s3:::amzn-s3-demo-bucket/*",
            "Condition": {
                "NumericGreaterThan": {
                    "s3:max-annotation-results": "100"
                }
            }
        }
    ]
}
```

## Examples: Amazon S3 condition keys for bucket operations

The following example policies show how you can use Amazon S3 specific condition keys for bucket operations.

### Example 1: Granting s3:GetObject permission with a condition on an IP address

You can give authenticated users permission to use the `s3:GetObject` action if the request originates from a specific range of IP addresses (for example, `` `192.0.2.*` ``), unless the IP address is one that you want to exclude (for example, `` `192.0.2.188` ``). In the `Condition` block, `IpAddress` and `NotIpAddress` are conditions, and each condition is provided a key-value pair for evaluation. Both of the key-value pairs in this example use the `aws:SourceIp` AWS wide key. To use this example policy, replace the `` `user input placeholders` `` with your own information.

###### Note

The `IPAddress` and `NotIpAddress` key values specified in the `Condition` block use CIDR notation, as described in RFC 4632. For more information, see [http://www.rfc-editor.org/rfc/rfc4632.txt](http://www.rfc-editor.org/rfc/rfc4632.txt).

anchor

```json
{
    "Version":"2012-10-17",
    "Id": "S3PolicyId1",
    "Statement": [
        {
            "Sid": "statement1",
            "Effect": "Allow",
            "Principal": "*",
            "Action":"s3:GetObject",
            "Resource": "arn:aws:s3:::amzn-s3-demo-bucket/*",
            "Condition" : {
                "IpAddress" : {
                    "aws:SourceIp": "192.0.2.0/24" 
                },
                "NotIpAddress" : {
                    "aws:SourceIp": "192.0.2.188/32" 
                } 
            } 
        } 
    ]
}
```

You can also use other AWS‐wide condition keys in Amazon S3 policies. For example, you can specify the `aws:SourceVpce` and `aws:SourceVpc` condition keys in bucket policies for VPC endpoints. For specific examples, see [Controlling access from VPC endpoints with bucket policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/example-bucket-policies-vpc-endpoint.html).

###### Note

For some AWS global condition keys, only certain resource types are supported. Therefore, check whether Amazon S3 supports the global condition key and resource type that you want to use, or if you'll need to use an Amazon S3 specific condition key instead. For a complete list of supported resource types and condition keys for Amazon S3, see [Actions, resources, and condition keys for Amazon S3](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazons3.html) in the *Service Authorization Reference*.

For more information about the permissions to S3 API operations by S3 resource types, see [Required permissions for Amazon S3 API operations](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-with-s3-policy-actions.html).

### Example 2: Getting a list of objects in a bucket with a specific prefix

You can use the `s3:prefix` condition key to limit the response of the [ListObjectsV2](https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListObjectsV2.html) API operation to key names with a specific prefix. If you are the bucket owner, you can use this condition key to restrict a user to list the contents of a specific prefix in the bucket. The `s3:prefix` condition key is useful if the objects in the bucket are organized by key name prefixes.

The Amazon S3 console uses key name prefixes to show a folder concept. Only the console supports the concept of folders; the Amazon S3 API supports only buckets and objects. For example, if you have two objects with the key names `` `public/object1.jpg` `` and `` `public/object2.jpg` ``, the console shows the objects under the `` `public` `` folder. In the Amazon S3 API, these are objects with prefixes, not objects in folders. For more information about using prefixes and delimiters to filter access permissions, see [Controlling access to a bucket with user policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/walkthrough1.html).

In the following scenario, the bucket owner and the parent account to which the user belongs are the same. So the bucket owner can use either a bucket policy or a user policy to grant access. For more information about other condition keys that you can use with the `ListObjectsV2` API operation, see [ListObjectsV2](https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListObjectsV2.html).

###### Note

If the bucket is versioning-enabled, to list the objects in the bucket, you must grant the `s3:ListBucketVersions` permission in the following policies, instead of the `s3:ListBucket` permission. The `s3:ListBucketVersions` permission also supports the `s3:prefix` condition key.

###### User policy

The following user policy grants the `s3:ListBucket` permission (see [ListObjectsV2](https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListObjectsV2.html)) with a `Condition` statement that requires the user to specify a prefix in the request with a value of `` `projects` ``. To use this example policy, replace the `` `user input placeholders` `` with your own information.

anchor

```json
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Sid":"statement1",
         "Effect":"Allow",
         "Action": "s3:ListBucket",
         "Resource":"arn:aws:s3:::amzn-s3-demo-bucket",
         "Condition" : {
             "StringEquals" : {
                 "s3:prefix": "projects" 
             }
          } 
       },
      {
         "Sid":"statement2",
         "Effect":"Deny",
         "Action": "s3:ListBucket",
         "Resource": "arn:aws:s3:::amzn-s3-demo-bucket",
         "Condition" : {
             "StringNotEquals" : {
                 "s3:prefix": "projects" 
             }
          } 
       }         
    ]
}
```

The `Condition` statement restricts the user to listing only object keys that have the `` `projects` `` prefix. The added explicit `Deny` statement denies the user from listing keys with any other prefix, no matter what other permissions the user might have. For example, it's possible that the user could get permission to list object keys without any restriction, either through updates to the preceding user policy or through a bucket policy. Because explicit `Deny` statements always override `Allow` statements, if the user tries to list keys other than those that have the `` `projects` `` prefix, the request is denied.

###### Bucket policy

If you add the `Principal` element to the above user policy, identifying the user, you now have a bucket policy, as shown in the following example. To use this example policy, replace the `` `user input 							placeholders` `` with your own information.

anchor

```json
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Sid":"statement1",
         "Effect":"Allow",
         "Principal": {
            "AWS": "arn:aws:iam::123456789012:user/bucket-owner"
         },  
         "Action":  "s3:ListBucket",
         "Resource": "arn:aws:s3:::amzn-s3-demo-bucket",
         "Condition" : {
             "StringEquals" : {
                 "s3:prefix": "projects" 
             }
          } 
       },
      {
         "Sid":"statement2",
         "Effect":"Deny",
         "Principal": {
            "AWS": "arn:aws:iam::123456789012:user/bucket-owner"
         },  
         "Action": "s3:ListBucket",
         "Resource": "arn:aws:s3:::amzn-s3-demo-bucket",
         "Condition" : {
             "StringNotEquals" : {
                 "s3:prefix": "projects"  
             }
          } 
       }         
    ]
}
```

###### Test the policy with the AWS CLI

You can test the policy using the following `list-object` AWS CLI command. In the command, you provide user credentials using the `--profile` parameter. For more information about setting up and using the AWS CLI, see [Developing with Amazon S3 using the AWS CLI](https://docs.aws.amazon.com/AmazonS3/latest/API/setup-aws-cli.html) in the *Amazon S3 API Reference*.

```
aws s3api list-objects --bucket amzn-s3-demo-bucket --prefix projects --profile AccountA
```

### Example 3: Setting the maximum number of keys

You can use the `s3:max-keys` condition key to set the maximum number of keys that a requester can return in a [ListObjectsV2](https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListObjectsV2.html) or [ListObjectVersions](https://docs.aws.amazon.com//AmazonS3/latest/API/API_ListObjectVersions.html) request. By default, these API operations return up to 1,000 keys. For a list of numeric condition operators that you can use with `s3:max-keys` and accompanying examples, see [Numeric Condition Operators](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_Numeric) in the *IAM User Guide*.