---
title: "Using AWS Identity and Access Management Access Analyzer - AWS Identity and Access Management"
source: "https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html"
author:
published:
created: 2026-07-19
description: "Learn about how AWS Identity and Access Management Access Analyzer analyzes resource-based policies to identify unintended access."
tags:
  - "clippings"
---
Using AWS Identity and Access Management Access Analyzer - AWS Identity and Access Management

AWS Identity and Access Management Access Analyzer provides the following capabilities:

## Identifying resources shared with an external entity

IAM Access Analyzer helps you identify the resources in your organization and accounts, such as Amazon S3 buckets or IAM roles, shared with an external entity. This lets you identify unintended access to your resources and data, which is a security risk. IAM Access Analyzer identifies resources shared with external principals by using logic-based reasoning to analyze the resource-based policies in your AWS environment. For each instance of a resource shared outside of your account, IAM Access Analyzer generates a finding. Findings include information about the access and the external principal granted to it. You can review findings to determine if the access is intended and safe or if the access is unintended and a security risk. In addition to helping you identify resources shared with an external entity, you can use IAM Access Analyzer findings to preview how your policy affects public and cross-account access to your resource before deploying resource permissions. The findings are organized in a visual summary dashboard. The dashboard highlights the split between public and cross-account access findings, and provides a breakdown of findings by resource type. To learn more about the dashboard, see [View the IAM Access Analyzer findings dashboard](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-dashboard.html).

###### Note

An external entity can be another AWS account, a root user, an IAM user or role, a federated user, an anonymous user, or another entity that you can use to create a filter. For more information, see [AWS JSON Policy Elements: Principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html).

When you enable IAM Access Analyzer, you create an analyzer for your entire organization or your account. The organization or account you choose is known as the zone of trust for the analyzer. The analyzer monitors all of the [supported resources](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html) within your zone of trust. Any access to resources by principals within your zone of trust is considered trusted. Once enabled, IAM Access Analyzer analyzes the policies applied to all of the supported resources in your zone of trust. After the first analysis, IAM Access Analyzer analyzes these policies periodically. If you add a new policy or change an existing policy, IAM Access Analyzer analyzes the new or updated policy within about 30 minutes.

When analyzing the policies, if IAM Access Analyzer identifies one that grants access to an external principal that isn't within your zone of trust, it generates a finding. Each finding includes details about the resource, the external entity with access to it, and the permissions granted so that you can take appropriate action. You can view the details included in the finding to determine whether the resource access is intentional or a potential risk that you should resolve. When you add a policy to a resource, or update an existing policy, IAM Access Analyzer analyzes the policy. IAM Access Analyzer also analyzes all resource-based policies periodically.

On rare occasions under certain conditions, IAM Access Analyzer does not receive notification of an added or updated policy, which can cause delays in generated findings. IAM Access Analyzer can take up to 6 hours to generate or resolve findings if you create or delete a multi-region access point associated with an Amazon S3 bucket, or update the policy for the multi-region access point. Also, if there is a delivery issue with AWS CloudTrail log delivery or resource control policy (RCP) restriction changes, the policy change does not trigger a rescan of the resource reported in the finding. When this happens, IAM Access Analyzer analyzes the new or updated policy during the next periodic scan, which is within 24 hours. If you want to confirm a change you make to a policy resolves an access issue reported in a finding, you can rescan the resource reported in a finding by using the **Rescan** link in the **Findings** details page, or by using the [`StartResourceScan`](https://docs.aws.amazon.com/access-analyzer/latest/APIReference/API_StartResourceScan.html) operation of the IAM Access Analyzer API. To learn more, see [Resolve IAM Access Analyzer findings](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-findings-remediate.html).

###### Important

For external access, IAM Access Analyzer analyzes only policies applied to resources in the same AWS Region where it's enabled. To monitor all resources in your AWS environment, you must create an external access analyzer to enable IAM Access Analyzer in each Region where you're using supported AWS resources.

For unused access, findings for the analyzer do not change based on Region. Creating an unused access analyzer in each Region where you have resources is not required.

IAM Access Analyzer analyzes the following resource types for external access:

- [Amazon Simple Storage Service buckets](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html#access-analyzer-s3)
- [Amazon Simple Storage Service directory buckets](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html#access-analyzer-s3-directory)
- [AWS Identity and Access Management roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html#access-analyzer-iam-role)
- [AWS Key Management Service keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html#access-analyzer-kms-key)
- [AWS Lambda functions and layers](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html#access-analyzer-lambda)
- [Amazon Simple Queue Service queues](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html#access-analyzer-sqs)
- [AWS Secrets Manager secrets](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html#access-analyzer-secrets-manager)
- [Amazon Simple Notification Service topics](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html#access-analyzer-sns)
- [Amazon Elastic Block Store volume snapshots](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html#access-analyzer-ebs)
- [Amazon Relational Database Service DB snapshots](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html#access-analyzer-rds-db)
- [Amazon Relational Database Service DB cluster snapshots](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html#access-analyzer-rds-db-cluster)
- [Amazon Elastic Container Registry repositories](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html#access-analyzer-ecr)
- [Amazon Elastic File System file systems](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html#access-analyzer-efs)
- [Amazon DynamoDB streams](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html#access-analyzer-ddb-stream)
- [Amazon DynamoDB tables](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-resources.html#access-analyzer-ddb-table)

## Identifying internal access to business-critical resources

For your selected business-critical resources, IAM Access Analyzer helps you identify which principals within your organization or account have access to them. This analysis supports implementing the principle of least privilege by ensuring that your specified resources can only be accessed by the intended principals within your organization.

Internal access analysis helps you:

- Determine which IAM users or roles within your account or organization can access your specified resources
- Understand access paths between principals and resources within your AWS environment
- Verify that your access controls are working as intended
- Review findings to determine if the access is intended and safe or if the access is unintended and presents a security risk
- Identify and remediate unintended access within your organization

IAM Access Analyzer analyzes the following resource types for internal access:

## Identifying unused access granted to IAM users and roles

IAM Access Analyzer helps you identify and review unused access in your AWS organization and accounts. IAM Access Analyzer continuously monitors all IAM roles and users in your AWS organization and accounts and generates findings for unused access. The findings highlight unused roles, unused access keys for IAM users, and unused passwords for IAM users. For active IAM roles and users, the findings provide visibility into unused services and actions.

IAM Access Analyzer reviews last accessed information for all roles in your AWS organization and accounts to help you identify unused access. IAM action last accessed information helps you identify unused actions for roles in your AWS accounts. For more information, see [Refine permissions in AWS using last accessed information](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_last-accessed.html).

The findings for external, internal, and unused access analyzers are organized into a visual summary dashboard. The dashboard highlights your AWS resources and AWS accounts that have the most findings and provides a breakdown of findings by type. For more information about the dashboard, see [View the IAM Access Analyzer findings dashboard](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-dashboard.html).

## Validating policies against AWS best practices

You can validate your policies against IAM [policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html) and [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) using the basic policy checks provided by IAM Access Analyzer policy validation. You can create or edit a policy using the AWS CLI, AWS API, or JSON policy editor in the IAM console. You can view policy validation check findings that include security warnings, errors, general warnings, and suggestions for your policy. These findings provide actionable recommendations that help you author policies that are functional and conform to AWS best practices. To learn more about validating policies using policy validation, see [Validate policies with IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-policy-validation.html).

## Validating policies against your specified security standards

You can validate your policies against your specified security standards using the IAM Access Analyzer custom policy checks. You can create or edit a policy using the AWS CLI, AWS API, or JSON policy editor in the IAM console. Through the console, you can check whether your updated policy grants new access compared to the existing version. Through AWS CLI and AWS API, you can also check specific IAM actions that you consider critical are not allowed by a policy. These checks highlight a policy statement that grants new access. You can update the policy statement and re-run the checks until the policy conform to your security standard. To learn more about validating policies using custom policy checks, see [Validate policies with IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html).

## Generating policies

IAM Access Analyzer analyzes your AWS CloudTrail logs to identify actions and services that have been used by an IAM entity (user or role) within your specified date range. It then generates an IAM policy that is based on that access activity. You can use the generated policy to refine an entity's permissions by attaching it to an IAM user or role. To learn more about generating policies using IAM Access Analyzer, see [IAM Access Analyzer policy generation](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-policy-generation.html).

## Pricing for IAM Access Analyzer

IAM Access Analyzer charges for unused access analysis based on the number of IAM roles and users analyzed per analyzer per month.

- You will be charged for each unused access analyzer that you create.
- Creating unused access analyzers across multiple Regions will result in you being charged for each analyzer.
- Service-linked roles aren't analyzed for unused access activity and they aren't included in the total number of IAM roles analyzed.

IAM Access Analyzer charges for internal access analysis based on the number of resources monitored per internal access analyzer per month.

IAM Access Analyzer charges for custom policy checks based on the number of API requests made to IAM Access Analyzer to check for new access.

For a complete list of prices and charges for IAM Access Analyzer, see [IAM Access Analyzer pricing](https://aws.amazon.com/iam/access-analyzer/pricing).

To see your bill, go to the **Billing and Cost Management Dashboard** in the [AWS Billing and Cost Management console](https://console.aws.amazon.com/billing/). Your bill contains links to usage reports that provide details about your bill. To learn more about AWS account billing, see the [AWS Billing User Guide](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/)

If you have questions concerning AWS billing, accounts, and events, [contact Support](https://aws.amazon.com/contact-us/).