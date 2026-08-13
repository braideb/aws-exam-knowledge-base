---
title: "Using EventBridge - Amazon Simple Storage Service"
source: "https://docs.aws.amazon.com/AmazonS3/latest/userguide/EventBridge.html"
author:
published:
created: 2026-07-18
description: "Receive notifications when specific Amazon S3 events such as object creation or deletion occur in an Amazon S3 bucket with EventBridge."
tags:
  - "clippings"
---
Using EventBridge - Amazon Simple Storage Service

Amazon S3 can send events to Amazon EventBridge whenever certain events happen in your bucket. Unlike other destinations, you don't need to select which event types you want to deliver. After EventBridge is enabled, all events below are sent to EventBridge. You can use EventBridge rules to route events to additional targets. The following lists the events Amazon S3 sends to EventBridge.

| Event type | Description |
| --- | --- |
| *Object Created* | An object was created.  The reason field in the event message structure indicates which S3 API was used to create the object: [PutObject](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObject.html), [POST Object](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectPOST.html), [CopyObject](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CopyObject.html), or [CompleteMultipartUpload](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CompleteMultipartUpload.html). |
| *Object Deleted (DeleteObject)*  *Object Deleted (Lifecycle expiration)* | An object was deleted.  When an object is deleted using an S3 API call, the reason field is set to DeleteObject. When an object is deleted by an S3 Lifecycle expiration rule, the reason field is set to Lifecycle Expiration. For more information, see [Expiring objects](https://docs.aws.amazon.com/AmazonS3/latest/userguide/lifecycle-expire-general-considerations.html).  When an unversioned object is deleted, or a versioned object is permanently deleted, the deletion-type field is set to Permanently Deleted. When a delete marker is created for a versioned object, the `deletion-type` field is set to Delete Marker Created. For more information, see [Deleting object versions from a versioning-enabled bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/DeletingObjectVersions.html). |
| *Object Restore Initiated* | An object restore was initiated from S3 Glacier Flexible Retrieval or S3 Glacier Deep Archive storage class or from S3 Intelligent-Tiering Archive Access or Deep Archive Access tier. For more information, see [Working with archived objects](https://docs.aws.amazon.com/AmazonS3/latest/userguide/archived-objects.html). |
| *Object Restore Completed* | An object restore was completed. |
| *Object Restore Expired* | The temporary copy of an object restored from S3 Glacier Flexible Retrieval or S3 Glacier Deep Archive expired and was deleted. |
| *Object Storage Class Changed* | An object was transitioned to a different storage class. For more information, see [Transitioning objects using Amazon S3 Lifecycle](https://docs.aws.amazon.com/AmazonS3/latest/userguide/lifecycle-transition-general-considerations.html). |
| *Object Access Tier Changed* | An object was transitioned to the S3 Intelligent-Tiering Archive Access tier or Deep Archive Access tier. For more information, see [Managing storage costs with Amazon S3 Intelligent-Tiering](https://docs.aws.amazon.com/AmazonS3/latest/userguide/intelligent-tiering.html). |
| *Object ACL Updated* | An object's access control list (ACL) was set using `PutObjectAcl`. An event is not generated when a request results in no change to an object's ACL. For more information, see [Access control list (ACL) overview](https://docs.aws.amazon.com/AmazonS3/latest/userguide/acl-overview.html). |
| *Object Tags Added* | A set of tags was added to an object using `PutObjectTagging`. For more information, see [Tagging your objects](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-tagging.html). |
| *Object Tags Deleted* | All tags were removed from an object using `DeleteObjectTagging`. For more information, see [Tagging your objects](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-tagging.html). |
| *Object Annotation Created* | An annotation was created or updated on an object using `PutObjectAnnotation`. For more information, see [Annotating your objects](https://docs.aws.amazon.com/AmazonS3/latest/userguide/annotations-overview.html). |
| *Object Annotation Removed* | An annotation was deleted from an object using `DeleteObjectAnnotation`. For more information, see [Annotating your objects](https://docs.aws.amazon.com/AmazonS3/latest/userguide/annotations-overview.html). |

###### Note

For more information about how Amazon S3 event types map to EventBridge event types, see [Amazon EventBridge mapping and troubleshooting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ev-mapping-troubleshooting.html).

You can use Amazon S3 Event Notifications with EventBridge to write rules that take actions when an event occurs in your bucket. For example, you can have it send you a notification. For more information, see [What is EventBridge?](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html) in the *Amazon EventBridge User Guide*.

For more information about the actions and data types you can interact with using the EventBridge API, see the [Amazon EventBridge API Reference](https://docs.aws.amazon.com/eventbridge/latest/APIReference/Welcome.html) in the *Amazon EventBridge API Reference*.

For information about pricing, see [Amazon EventBridge pricing](https://aws.amazon.com/eventbridge/pricing).