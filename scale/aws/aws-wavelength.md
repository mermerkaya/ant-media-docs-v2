---
title: AWS Wavelength Deployment
description: Deploy Ant Media Server at AWS Wavelength zones for ultra-low latency streaming to 5G mobile devices under 150ms.
keywords: [AWS Wavelength, 5G edge streaming, ultra-low latency, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 6
---

# Deploy AMS at AWS Wavelength

AWS Wavelength deploys AWS compute and storage at the edge of 5G carrier networks, enabling ultra-low latency connections to mobile devices. Deploying AMS on Wavelength can achieve sub-150ms latency under Wavelength conditions.

```mermaid
flowchart LR
    MOB["5G Mobile Device"]
    WZ["AWS Wavelength Zone\n(Carrier Network Edge)"]
    AMS["Ant Media Server\non Wavelength EC2"]
    VPC["AWS Region VPC\nServices"]

    MOB -->|"5G Network"| WZ
    WZ --> AMS
    AMS <--> VPC
```

## Key Considerations

- **No Elastic Load Balancer in Wavelength Zones**: The CloudFormation template for Wavelength includes a special Nginx load balancer that listens to the Auto Scaling group and updates its configuration automatically — replacing the standard ALB that is unavailable in Wavelength zones.
- **SSL Configuration**: Wavelength instances require special SSL setup because they are at the carrier edge.
- **STUN Server Configuration**: WebRTC ICE candidates must be configured for the Wavelength topology.
- Requires **AMS v2.4.1 or later**.

## Deployment Options

Two CloudFormation-based deployment patterns are available for Wavelength:

### Standalone Server Deployment

Deploy a single AMS instance in a Wavelength Zone for development or low-traffic scenarios. See the [Standalone Wavelength deployment guide](https://antmedia.io/docs/guides/clustering-and-scaling/aws/aws-wavelength-standalone-deployment/).

### Auto-Scalable Cluster Deployment

Deploy an Origin + Edge cluster with Auto Scaling in Wavelength Zones. The custom Nginx load balancer automatically tracks Auto Scaling group membership. See the [Cluster Wavelength deployment guide](https://antmedia.io/docs/guides/clustering-and-scaling/aws/aws-wavelength-cluster-deployment/).

## Related Guides

- [SSL Setup](https://antmedia.io/docs/guides/installing-on-linux/setting-up-ssl/)
- [Configure STUN/TURN Addresses](https://antmedia.io/docs/guides/configuration-and-testing/configuring-stun-turn-addresses/)
