[UID2 Documentation](../../README.md) > v1 > Integration Guides > Advertiser/Data Provider Integration Guide

# Overview

The following describes steps that Data Providers/Advertisers can take to map PII (Email or SHA256 of it) to UID2 for targeting and audience building purposes. 

```mermaid
  sequenceDiagram
    participant Data Provider
    participant UID Service
    participant DSP
    Data Provider->>UID Service: 1. Map PII to UID2
    UID Service->>Data Provider: Mapped UID2s along with Bucket Info
    Data Provider-->>DSP: 2. Send UID2 based Audience
    loop Refresh
       Data Provider->>UID Service: 3. Check for Bucket Updates
    end
    Data Provider->>UID Service: 4. Incremental Update UID2.
```

# Steps

## 1. Map PII to UID2.

Use the [/identity/map](../endpoints/post-identity-map.md) enpoint to map PII to UID2. The UID2 returned that can be used to target audiences on relavant DSPs, other use cases.

## 2. Send UID2 based Audience
This will be a DSP specific endpoint/mechanism and is out of scope for this document. Follow DSP specific Integration for sending UID2 based audience.

## 3. Check for Bucket Updates

By very design, the UID2 is an ID for that user in that moment in time. A user's UID2 can and will rotate once a year. One can choose to either blanketly update their segments on daily basis or can implement incremental updates by querying bucket rotation status.

Bucket status endpoint [/identity/buckets](../endpoints/get-identity-buckets.md) can return the buckets that have their associated UID2s rotated since the given timestamp.

## 4. Incremental Update UID2.

Leveraging Steps 1 and 3, a system can continously update its UID2 based audiences to the DSP. The response from Step 1 contains mapped UID2 and corresponding "bucket_id" associated with it. While UID2 can rotate, the bucket_id will stay constant for the given user.

From Step 1. the system can store the Mapping between PII and Bucket ID along with last updated timestamp. e.g.

```
PIIIdentifier, BucketId, Updated Timestamp
```

By querying the Bucket info described in Step 3, the system can repeat step 1 and 2 for the IDs associated with the updated buckets.

# Frequently Asked Questions
### Q: How does a holder of UID2 know when to refresh the UID2 due to salt rotation?
Metadata supplied with the UID2 generation request indicates the salt bucket used for generating the UID2. Salt buckets are persistent and assigned to the underlying PII. Use the API provided to return which salt buckets rotated since a given timestamp. The returned rotated salt buckets inform the UID2 holder which UID2s to refresh. This workflow typically applies to data providers. 

### Q: How often should IDs be refreshed for Incemetal Updates?
The recommended cadence for updating Audiences is daily. 

### Q: How should i generate the SHA256 of PII for mapping?
The system should follow the email normalization rules (described in Overview document) and hash without salting. The value needs to be base64 encoded before it can be sent.

