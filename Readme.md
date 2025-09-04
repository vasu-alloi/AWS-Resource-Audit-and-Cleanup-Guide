# AWS Resource Audit and Cleanup Guide

## Overview
This guide provides a comprehensive approach to audit AWS resources, identify unused/idle resources, and safely clean them up to optimize costs.

## Prerequisites
- AWS CLI configured with appropriate permissions
- Python 3.x with boto3 library
- Backup strategy in place
- List of critical resources that should never be deleted

## Phase 1: Resource Discovery and Inventory

### 1.1 EC2 Resources Audit
```bash
# List all EC2 instances with their state and launch time
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,LaunchTime,InstanceType,Tags[?Key==`Name`].Value|[0]]' --output table

# Find stopped instances older than 7 days
aws ec2 describe-instances --filters "Name=instance-state-name,Values=stopped" --query 'Reservations[*].Instances[?LaunchTime<=`'$(date -d '7 days ago' -Iseconds)'`].[InstanceId,LaunchTime,Tags[?Key==`Name`].Value|[0]]' --output table
```
<img width="1855" height="584" alt="image" src="https://github.com/user-attachments/assets/6dbfa25d-ed07-4091-81f5-f3f554fdc959" />

<img width="1836" height="255" alt="image" src="https://github.com/user-attachments/assets/7654a533-1cf5-40b3-ba3e-ab1319ff5e71" />


### 1.2 EBS Volumes Audit
```bash
# List unattached EBS volumes
aws ec2 describe-volumes --filters "Name=status,Values=available" --query 'Volumes[*].[VolumeId,Size,VolumeType,CreateTime,Tags[?Key==`Name`].Value|[0]]' --output table

# List volumes attached to stopped instances
aws ec2 describe-volumes --query 'Volumes[?Attachments[0].State==`attached`]' --output table
```
<img width="1831" height="988" alt="image" src="https://github.com/user-attachments/assets/95386176-2d9a-46cc-a5b2-356e43f4ccda" />

<img width="1776" height="907" alt="image" src="https://github.com/user-attachments/assets/83daac29-54df-484f-970e-c374b0f5ace1" />
# EBS Volumes Analysis

## Summary
- **Total Volumes**: 50
- **Unattached Volumes (None tag)**: 35
- **Attached Volumes (Kubernetes PVCs)**: 15
- **Total Storage (Unattached)**: 258 GB
- **Estimated Monthly Cost (Unattached)**: ~$25.80 USD

## Detailed Volume Inventory

| Volume ID | Size (GB) | Type | Creation Date | Status | Age (Days) | Monthly Cost |
|-----------|-----------|------|---------------|---------|------------|--------------|
| vol-07cea0c1a466d0097 | 8 | gp2 | 2025-07-17T12:50:22 | Unattached | 48 | $0.80 |
| vol-0805b513a4f183c7e | 8 | gp2 | 2025-07-24T04:27:05 | Attached (K8s) | 41 | - |
| vol-0c0b759f668ca6290 | 8 | gp2 | 2025-07-24T04:27:43 | Attached (K8s) | 41 | - |
| vol-04c53045ed32a4a89 | 8 | gp2 | 2025-07-26T00:51:24 | Unattached | 39 | $0.80 |
| vol-093d89f0859eb4463 | 50 | gp2 | 2024-12-23T21:01:58 | Unattached | 254 | $5.00 |
| vol-039d892cab63d2257 | 2 | gp2 | 2025-07-28T11:37:33 | Unattached | 37 | $0.20 |
| vol-01fd793febba56e2b | 30 | gp2 | 2025-03-26T21:22:25 | Unattached | 161 | $3.00 |
| vol-08ff211ccbead4b62 | 8 | gp2 | 2025-07-24T04:27:05 | Attached (K8s) | 41 | - |
| vol-04f13d74b5f573521 | 2 | gp2 | 2025-07-24T04:27:05 | Attached (K8s) | 41 | - |
| vol-0d45a3290f016342b | 8 | gp2 | 2025-07-17T12:46:14 | Unattached | 48 | $0.80 |
| vol-008e5dbb35d667ec8 | 10 | gp2 | 2025-07-16T10:20:18 | Unattached | 49 | $1.00 |
| vol-0de8cd08cb4257306 | 8 | gp2 | 2025-07-26T00:52:01 | Unattached | 39 | $0.80 |
| vol-02801e2e1c9bfda49 | 8 | gp2 | 2025-06-03T07:23:22 | Unattached | 92 | $0.80 |
| vol-0da8c3c626b5644ad | 1 | gp2 | 2025-07-24T04:27:15 | Attached (K8s) | 41 | - |
| vol-022082ae3571c9eb4 | 2 | gp2 | 2025-07-17T12:49:18 | Unattached | 48 | $0.20 |
| vol-044b3301252f5b75f | 8 | gp2 | 2025-07-24T04:28:16 | Attached (K8s) | 41 | - |
| vol-0117a75dd80657a2c | 1 | gp2 | 2025-07-17T12:49:17 | Unattached | 48 | $0.10 |
| vol-0ae83c391f6fdad0a | 2 | gp2 | 2025-07-26T00:56:01 | Unattached | 39 | $0.20 |
| vol-0008caf7aea9d356d | 2 | gp2 | 2025-07-17T12:49:27 | Unattached | 48 | $0.20 |
| vol-029ec7328a13f7c33 | 30 | gp3 | 2025-03-26T21:22:25 | Unattached | 161 | $2.40 |
| vol-06739259fed902a86 | 2 | gp2 | 2025-07-24T04:27:27 | Attached (K8s) | 41 | - |
| vol-089107da9ac877805 | 10 | gp2 | 2025-07-26T00:50:45 | Unattached | 39 | $1.00 |
| vol-06f2986a734639e28 | 8 | gp2 | 2025-07-26T00:50:45 | Unattached | 39 | $0.80 |
| vol-0488a103b3c017fb8 | 8 | gp2 | 2025-05-14T08:12:45 | Unattached | 112 | $0.80 |
| vol-0a12d50ece925e6b9 | 8 | gp2 | 2025-05-14T08:13:24 | Unattached | 112 | $0.80 |
| vol-0132956d13ad13934 | 8 | gp3 | 2025-06-09T16:30:51 | Unattached | 85 | $0.64 |
| vol-0651754806cc9a18e | 2 | gp2 | 2025-07-24T04:27:15 | Attached (K8s) | 41 | - |
| vol-0a2eac1c3c2e942d1 | 8 | gp2 | 2025-09-03T12:59:42 | Unattached | 0 | $0.80 |
| vol-02630548dd28f5eed | 1 | gp2 | 2025-07-17T12:46:14 | Unattached | 48 | $0.10 |
| vol-033cff8c6e8bfd92c | 1 | gp2 | 2025-07-17T12:49:27 | Unattached | 48 | $0.10 |
| vol-0f1fd8c3a74f1f89e | 30 | gp3 | 2025-03-26T21:22:25 | Unattached | 161 | $2.40 |
| vol-0d83f93c1d3490c73 | 8 | gp3 | 2025-06-03T07:22:49 | Unattached | 92 | $0.64 |
| vol-003c5e8adc62b2198 | 8 | gp2 | 2025-06-09T16:29:46 | Unattached | 85 | $0.80 |
| vol-0861f263e3762f66f | 1 | gp2 | 2025-07-26T00:56:01 | Unattached | 39 | $0.10 |
| vol-03dba130b9ec9540a | 1 | gp2 | 2025-07-17T12:46:14 | Unattached | 48 | $0.10 |
| vol-0fb7497e2bc273706 | 10 | gp2 | 2025-07-17T12:46:14 | Unattached | 48 | $1.00 |
| vol-0b0c4f7bc2d7966af | 2 | gp2 | 2025-07-26T00:50:45 | Unattached | 39 | $0.20 |
| vol-0beee7366b47395ae | 1 | gp2 | 2025-07-24T04:27:05 | Attached (K8s) | 41 | - |
| vol-03974663b8427f914 | 2 | gp2 | 2025-07-24T04:27:05 | Attached (K8s) | 41 | - |
| vol-05b8525cce2b398d4 | 8 | gp2 | 2025-07-26T00:50:45 | Unattached | 39 | $0.80 |
| vol-040cb9ec4d4de93a0 | 1 | gp2 | 2025-07-24T04:27:27 | Attached (K8s) | 41 | - |
| vol-0e0bfb8eb8ae862c9 | 1 | gp2 | 2025-07-28T11:36:42 | Unattached | 37 | $0.10 |
| vol-09b0750b2b79a1d52 | 1 | gp2 | 2025-07-01T10:28:01 | Unattached | 64 | $0.10 |
| vol-01758935e0844daa1 | 2 | gp2 | 2025-07-17T12:46:14 | Unattached | 48 | $0.20 |
| vol-0bbec92ee8ba45bc7 | 1 | gp2 | 2025-07-17T12:46:14 | Unattached | 48 | $0.10 |
| vol-04bf081f2f778b66a | 8 | gp3 | 2025-06-09T16:22:14 | Unattached | 85 | $0.64 |
| vol-016d7ddf9196d6ca8 | 10 | gp2 | 2025-07-24T04:27:05 | Attached (K8s) | 41 | - |
| vol-01b0aad8c20ad4f33 | 8 | gp2 | 2025-07-17T12:49:17 | Unattached | 48 | $0.80 |
| vol-02e139fefad31041f | 8 | gp2 | 2025-07-17T12:49:50 | Unattached | 48 | $0.80 |
| vol-060952b96698428d7 | 2 | gp2 | 2025-07-17T12:46:14 | Unattached | 48 | $0.20 |
| vol-081a272e5c4979ca3 | 2 | gp2 | 2025-07-28T11:36:42 | Unattached | 37 | $0.20 |
| vol-01dc1c9c63b8987d0 | 1 | gp2 | 2025-07-28T11:37:33 | Unattached | 37 | $0.10 |

## Cleanup Recommendations

### High Priority (Immediate Cleanup)
1. **vol-093d89f0859eb4463** - 50GB, 254 days old, $5.00/month
2. **vol-01fd793febba56e2b** - 30GB, 161 days old, $3.00/month
3. **vol-029ec7328a13f7c33** - 30GB (gp3), 161 days old, $2.40/month
4. **vol-0f1fd8c3a74f1f89e** - 30GB (gp3), 161 days old, $2.40/month

### Medium Priority (30+ days old)
- Multiple 8GB volumes created in May-July 2025
- Total potential savings: ~$8-12/month

### Low Priority (Recent volumes)
- vol-0a2eac1c3c2e942d1 (created today)
- Multiple volumes from July 28, 2025

## Action Items
1. **Verify dependencies** for high-priority volumes before deletion
2. **Create snapshots** for any volumes containing important data
3. **Delete unattached volumes** older than 30 days after verification
4. **Consider converting** remaining GP2 volumes to GP3 for cost savings

## Cost Impact
- **Current monthly waste**: ~$25.80 USD
- **Annual savings potential**: ~$309.60 USD
### 1.3 Snapshots Audit
```bash
# List snapshots older than 30 days (adjust date as needed)
aws ec2 describe-snapshots --owner-ids self --query 'Snapshots[?StartTime<=`'$(date -d '30 days ago' -Iseconds)'`].[SnapshotId,VolumeId,StartTime,Description]' --output table
```
<img width="1832" height="240" alt="image" src="https://github.com/user-attachments/assets/1f2604a7-1e41-4234-a165-0e4b20edf589" />

### No volumes attached to this snapshots:

<img width="1610" height="280" alt="image" src="https://github.com/user-attachments/assets/a5f9fdc3-3564-46a7-aaa6-1c254e8acdd9" />


### 1.4 AMI Audit
```bash
# List AMIs owned by account
aws ec2 describe-images --owners self --query 'Images[*].[ImageId,Name,CreationDate,State]' --output table

# Check AMI usage (requires checking launch configurations, launch templates, etc.)
aws ec2 describe-launch-templates --query 'LaunchTemplates[*].[LaunchTemplateId,LaunchTemplateName]' --output table
```

<img width="1715" height="451" alt="image" src="https://github.com/user-attachments/assets/7d8a0843-0c16-473a-9308-2d5b84b7ef1d" />


### 1.5 Elastic IP Audit
```bash
# List unassociated Elastic IPs
aws ec2 describe-addresses --query 'Addresses[?AssociationId==null].[PublicIp,AllocationId,Domain]' --output table
```
<img width="1834" height="249" alt="image" src="https://github.com/user-attachments/assets/83a47c3f-153c-475b-b496-30c47907bea4" />


### 1.6 Load Balancers Audit
```bash
# Classic Load Balancers
aws elb describe-load-balancers --query 'LoadBalancerDescriptions[*].[LoadBalancerName,Instances[0].InstanceId,CreatedTime]' --output table

# Application/Network Load Balancers
aws elbv2 describe-load-balancers --query 'LoadBalancers[*].[LoadBalancerName,State.Code,CreatedTime]' --output table

# Check target health
aws elbv2 describe-target-groups --query 'TargetGroups[*].[TargetGroupName,TargetType]' --output table
```
### ClassicLoadBalancer:
<img width="1827" height="182" alt="image" src="https://github.com/user-attachments/assets/fd701bfe-9892-4db9-85fe-a16865750be5" />
### Application&NetworkLoadBalancer:

<img width="1853" height="217" alt="image" src="https://github.com/user-attachments/assets/f09b1b07-4e1a-476d-85f0-73816d7e2869" />
### TargetGroups:
<img width="1813" height="985" alt="image" src="https://github.com/user-attachments/assets/44a88de9-d493-455b-ad59-ab021b5a15b5" />


### 1.7 S3 Buckets Audit
```bash
# List all buckets
aws s3 ls

# Check bucket sizes and last modified dates
aws s3 ls --recursive s3://bucket-name --summarize

# Check bucket policies and access logs
aws s3api get-bucket-policy --bucket bucket-name
aws s3api get-bucket-logging --bucket bucket-name
```
### ListOfBuckets:
<img width="1376" height="464" alt="image" src="https://github.com/user-attachments/assets/47c7acc0-ce5d-4bf8-9d92-bb413e2c6382" />
### ListEmptyBuckets
<img width="1103" height="268" alt="image" src="https://github.com/user-attachments/assets/8a707948-2fea-4125-846a-d59d492af34c" />
# S3 Bucket Analysis Report

## Summary
- **Total Buckets**: 16
- **Empty Buckets**: 4 (25%)
- **Buckets with Objects**: 12 (75%)
- **Total Objects Across All Buckets**: ~3,378 objects
- **CloudWatch Metrics**: Not enabled for any bucket

## Detailed Bucket Inventory

| Bucket Name | Status | Object Count | CloudWatch Metrics | Cleanup Priority | Action Recommended |
|-------------|---------|--------------|-------------------|------------------|-------------------|
| demo.opshealth.io | Active | 106 | Not Available | Low | Review content, enable metrics |
| dev.opshealth.io | Active | 106 | Not Available | Low | Review content, enable metrics |
| **devops.opshealth.io** | **Empty** | **0** | **Not Available** | **High** | **Delete if unused** |
| jenkins-test.opshealth.io | Active | 100 | Not Available | Medium | Verify if test env is active |
| onprem-rafay | Active | 101 | Not Available | Low | Review content, enable metrics |
| opshealth-demo | Active | 113 | Not Available | Low | Review content, enable metrics |
| opshealth-dev | Active | 209 | Not Available | Low | Review content, enable metrics |
| opshealth-local-test | Active | 3 | Not Available | Medium | Verify if test env is active |
| opshealth-quickwit-dev | Active | 1 | Not Available | Medium | Verify if dev env is active |
| opshealth-test | Active | 102 | Not Available | Medium | Verify if test env is active |
| **qa.opshealth.io** | **Empty** | **0** | **Not Available** | **High** | **Delete if unused** |
| **sandbox.opshealth.io** | **Empty** | **0** | **Not Available** | **High** | **Delete if unused** |
| service-catalog-logos | Active | 3 | Not Available | Low | Likely in use for UI |
| **stage.opshealth.io** | **Empty** | **0** | **Not Available** | **High** | **Delete if unused** |
| test-timescaledb-1 | Active | 2,040 | Not Available | Low | Database backups - keep |
| test.opshealth.io | Active | 100 | Not Available | Medium | Verify if test env is active |

## Cost Analysis

### Empty Buckets (Immediate Savings)
| Bucket Name | Monthly Cost | Annual Cost | Status |
|-------------|--------------|-------------|---------|
| devops.opshealth.io | $0.00* | $0.00* | Ready for deletion |
| qa.opshealth.io | $0.00* | $0.00* | Ready for deletion |
| sandbox.opshealth.io | $0.00* | $0.00* | Ready for deletion |
| stage.opshealth.io | $0.00* | $0.00* | Ready for deletion |

*Empty buckets have no storage costs but may have minimal request/management costs

### Environment-Based Analysis
| Environment | Bucket Count | Total Objects | Status |
|-------------|--------------|---------------|---------|
| Production | 1 | 106 | Active (demo.opshealth.io) |
| Development | 4 | 319 | Mixed usage |
| Testing | 5 | 2,245 | Mixed usage |
| Staging | 1 | 0 | **Empty - Delete candidate** |
| QA | 1 | 0 | **Empty - Delete candidate** |
| Sandbox | 1 | 0 | **Empty - Delete candidate** |
| Other | 4 | 708 | Active |

## Cleanup Recommendations

### 🔴 High Priority - Immediate Action
1. **Empty Environment Buckets** (4 buckets)
   - `devops.opshealth.io` - Empty DevOps bucket
   - `qa.opshealth.io` - Empty QA environment
   - `sandbox.opshealth.io` - Empty sandbox environment  
   - `stage.opshealth.io` - Empty staging environment

### 🟡 Medium Priority - Review Required
2. **Test Environment Buckets** (4 buckets)
   - `jenkins-test.opshealth.io` - Verify Jenkins usage
   - `opshealth-local-test` - Check local development needs
   - `opshealth-quickwit-dev` - Verify Quickwit development
   - `test.opshealth.io` - General test environment

### 🟢 Low Priority - Monitor
3. **Active Production/Development** (8 buckets)
   - All contain significant objects
   - Enable CloudWatch metrics for monitoring
   - Review lifecycle policies


### 1.8 RDS Resources Audit
```bash
# List RDS instances
aws rds describe-db-instances --query 'DBInstances[*].[DBInstanceIdentifier,DBInstanceStatus,Engine,DBInstanceClass,InstanceCreateTime]' --output table

# List RDS snapshots
aws rds describe-db-snapshots --query 'DBSnapshots[*].[DBSnapshotIdentifier,DBInstanceIdentifier,SnapshotCreateTime,Status]' --output table
```

### 1.9 Lambda Functions Audit
```bash
# List Lambda functions with last modified date
aws lambda list-functions --query 'Functions[*].[FunctionName,Runtime,LastModified,CodeSize]' --output table
```

### 1.10 Security Groups and Key Pairs
```bash
# List security groups
aws ec2 describe-security-groups --query 'SecurityGroups[*].[GroupId,GroupName,Description]' --output table

# List key pairs
aws ec2 describe-key-pairs --query 'KeyPairs[*].[KeyName,KeyFingerprint]' --output table
```

## Phase 2: Idle Resource Identification

### 2.1 CloudWatch Metrics Analysis
Use CloudWatch to identify truly idle resources:

```bash
# Check CPU utilization for EC2 instances (last 7 days)
aws cloudwatch get-metric-statistics --namespace AWS/EC2 --metric-name CPUUtilization --dimensions Name=InstanceId,Value=i-1234567890abcdef0 --statistics Average --start-time $(date -d '7 days ago' -Iseconds) --end-time $(date -Iseconds) --period 86400
```

### 2.2 VPC Flow Logs Analysis
Enable and analyze VPC Flow Logs to identify unused network resources.

### 2.3 Access Logs Analysis
- S3 access logs
- ELB access logs
- CloudTrail logs

## Phase 3: Automated Audit Script

### 3.1 Python Audit Script
```python
#!/usr/bin/env python3
import boto3
from datetime import datetime, timedelta
import json

class AWSResourceAuditor:
    def __init__(self):
        self.ec2 = boto3.client('ec2')
        self.s3 = boto3.client('s3')
        self.rds = boto3.client('rds')
        self.elb = boto3.client('elb')
        self.elbv2 = boto3.client('elbv2')
        self.lambda_client = boto3.client('lambda')
        self.cloudwatch = boto3.client('cloudwatch')
    
    def audit_ec2_instances(self):
        """Audit EC2 instances for stopped instances older than 7 days"""
        instances = self.ec2.describe_instances()
        idle_instances = []
        
        cutoff_date = datetime.now() - timedelta(days=7)
        
        for reservation in instances['Reservations']:
            for instance in reservation['Instances']:
                if instance['State']['Name'] == 'stopped':
                    launch_time = instance['LaunchTime'].replace(tzinfo=None)
                    if launch_time < cutoff_date:
                        idle_instances.append({
                            'InstanceId': instance['InstanceId'],
                            'LaunchTime': launch_time,
                            'State': instance['State']['Name']
                        })
        
        return idle_instances
    
    def audit_ebs_volumes(self):
        """Audit unattached EBS volumes"""
        volumes = self.ec2.describe_volumes(
            Filters=[{'Name': 'status', 'Values': ['available']}]
        )
        
        unattached_volumes = []
        for volume in volumes['Volumes']:
            unattached_volumes.append({
                'VolumeId': volume['VolumeId'],
                'Size': volume['Size'],
                'VolumeType': volume['VolumeType'],
                'CreateTime': volume['CreateTime']
            })
        
        return unattached_volumes
    
    def audit_elastic_ips(self):
        """Audit unassociated Elastic IPs"""
        addresses = self.ec2.describe_addresses()
        unassociated_eips = []
        
        for address in addresses['Addresses']:
            if 'AssociationId' not in address:
                unassociated_eips.append({
                    'PublicIp': address['PublicIp'],
                    'AllocationId': address['AllocationId']
                })
        
        return unassociated_eips
    
    def generate_report(self):
        """Generate comprehensive audit report"""
        report = {
            'audit_date': datetime.now().isoformat(),
            'idle_ec2_instances': self.audit_ec2_instances(),
            'unattached_ebs_volumes': self.audit_ebs_volumes(),
            'unassociated_elastic_ips': self.audit_elastic_ips()
        }
        
        return report

if __name__ == "__main__":
    auditor = AWSResourceAuditor()
    report = auditor.generate_report()
    
    with open(f'aws_audit_report_{datetime.now().strftime("%Y%m%d_%H%M%S")}.json', 'w') as f:
        json.dump(report, f, indent=2, default=str)
    
    print("Audit complete. Report saved.")
```

## Phase 4: Cost Analysis

### 4.1 AWS Cost Explorer
- Use Cost Explorer to identify high-cost resources
- Set up cost anomaly detection
- Create cost allocation tags

### 4.2 AWS Trusted Advisor
- Review Trusted Advisor recommendations for cost optimization
- Check for idle resources and right-sizing opportunities

## Phase 5: Safe Deletion Process

### 5.1 Pre-Deletion Checklist
- [ ] Create backups of critical data
- [ ] Review dependencies between resources
- [ ] Get approval from stakeholders
- [ ] Schedule maintenance window if needed
- [ ] Test deletion in non-production first

### 5.2 Deletion Scripts
```bash
# Example: Safe EC2 instance termination
aws ec2 create-snapshot --volume-id vol-1234567890abcdef0 --description "Backup before cleanup"
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# Example: EBS volume deletion
aws ec2 delete-volume --volume-id vol-1234567890abcdef0

# Example: Elastic IP release
aws ec2 release-address --allocation-id eipalloc-1234567890abcdef0
```

### 5.3 Rollback Plan
- Document all deletion actions
- Keep snapshots/backups for recovery
- Have a rollback procedure ready

## Phase 6: Monitoring and Prevention

### 6.1 Set up CloudWatch Alarms
- CPU utilization alerts
- Network activity alerts
- Storage usage alerts

### 6.2 Implement Resource Tagging Strategy
- Mandatory tags: Environment, Owner, Project, Purpose
- Lifecycle tags: Creation date, Review date, Expiry date

### 6.3 Automation
- Use AWS Config rules for compliance
- Set up Lambda functions for automatic cleanup
- Implement Infrastructure as Code (CloudFormation/Terraform)

## Cost Optimization Recommendations

### Immediate Actions
1. Release unassociated Elastic IPs
2. Delete unattached EBS volumes (after verification)
3. Terminate long-stopped EC2 instances
4. Remove unused security groups and key pairs

### Medium-term Actions
1. Right-size EC2 instances based on utilization
2. Convert GP2 to GP3 EBS volumes
3. Implement S3 lifecycle policies
4. Schedule EC2 instances for non-production workloads

### Long-term Actions
1. Implement Reserved Instances/Savings Plans
2. Use Spot instances for fault-tolerant workloads
3. Set up automated resource lifecycle management
4. Regular quarterly audits

## Security Considerations
- Ensure deletion permissions are properly scoped
- Log all deletion activities
- Review IAM policies for resource creation
- Implement approval workflows for resource deletion

## Report Template
```
AWS Resource Audit Report
========================
Date: [Date]
Account ID: [Account ID]
Auditor: [Name]

Executive Summary:
- Total resources audited: [Number]
- Idle resources identified: [Number]
- Estimated monthly savings: $[Amount]

Detailed Findings:
[Resource Type] - [Count] - [Estimated Cost]

Recommendations:
1. [Action] - [Priority] - [Savings]
2. [Action] - [Priority] - [Savings]

Next Steps:
1. Review and approve deletions
2. Create backups where necessary
3. Execute cleanup plan
4. Implement monitoring
```

## Important Notes
- Always test in non-production environments first
- Consider dependencies between resources
- Some resources may have termination protection enabled
- Be aware of data retention requirements
- Document all actions for audit purposes
