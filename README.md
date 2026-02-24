# Zero-Worker Management Cluster Test

This folder contains all files and documentation related to testing EKS Capabilities on a cluster with zero worker nodes.

## Test Overview

**Objective:** Determine whether AWS EKS Capabilities (ACK, ArgoCD, kro) can function on an EKS cluster with no worker nodes.

**Result:** ✅ CONFIRMED - EKS Capabilities work perfectly on zero-node clusters and can manage workload clusters.

## Test Clusters

1. **Management Cluster (Zero Nodes)**
   - Name: `eks-mgmt-cluster-ack-test-only`
   - Region: us-west-2
   - Worker nodes: 0
   - Purpose: Test ACK capability functionality

2. **Workload Cluster (Created by ACK)**
   - Name: `eks-workload-cluster-test-only`
   - Region: us-west-2
   - Created by: ACK capability running on zero-node management cluster
   - Purpose: Prove ACK can manage workload clusters from zero-node cluster

## Files in This Folder

- `EKS-CAPABILITIES-ZERO-NODE-TEST-RESULTS.md` - Complete test results and findings
- `ack-workload-cluster.yaml` - ACK Cluster CR for creating workload cluster
- `ack-eks-policy.json` - IAM policy for ACK capability to manage EKS
- `eks-cluster-trust-policy.json` - Trust policy for EKS cluster service role

## Key Findings

1. **Zero nodes required** - EKS Capabilities run in AWS infrastructure, not in your cluster
2. **Cost savings** - ~27% reduction ($36/month) vs traditional 2-node management cluster
3. **Prerequisites** - Cluster must use `API` or `API_AND_CONFIG_MAP` authentication mode
4. **Limitations** - Cannot run in-cluster workloads (Terraform operator, custom controllers, etc.)

## Test Status

- ✅ Zero-node management cluster created
- ✅ ACK capability enabled and ACTIVE
- ✅ Workload cluster created successfully via ACK
- ✅ Workload cluster status: ACTIVE
- ✅ Test complete - All objectives achieved
- ✅ Cleanup complete - All resources deleted

## Cleanup Performed

All test resources have been successfully cleaned up on February 24, 2026:

### Deletion Order (Important!)

1. **Workload cluster** - Must be deleted first
   ```bash
   aws eks delete-cluster \
     --name eks-workload-cluster-test-only \
     --region us-west-2
   ```
   Status: ✅ Deleted

2. **ACK capability** - Delete after workload cluster deletion initiated
   ```bash
   aws eks delete-capability \
     --cluster-name eks-mgmt-cluster-ack-test-only \
     --region us-west-2 \
     --capability-name ack-test-capability
   ```
   Status: ✅ Deleted
   Note: Capability deletion took ~45 seconds

3. **Management cluster** - Delete after capability is fully deleted
   ```bash
   aws eks delete-cluster \
     --name eks-mgmt-cluster-ack-test-only \
     --region us-west-2
   ```
   Status: ✅ Deleted

4. **IAM roles and policies** - Clean up after clusters are deleted
   
   a. Detach policies from ACKCapabilityTestRole:
   ```bash
   aws iam detach-role-policy \
     --role-name ACKCapabilityTestRole \
     --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
   
   aws iam detach-role-policy \
     --role-name ACKCapabilityTestRole \
     --policy-arn arn:aws:iam::833542146025:policy/ACKEKSPolicy
   ```
   
   b. Delete inline policy and role:
   ```bash
   aws iam delete-role-policy \
     --role-name ACKCapabilityTestRole \
     --policy-name ACKEKSPolicy
   
   aws iam delete-role --role-name ACKCapabilityTestRole
   ```
   Status: ✅ Deleted
   
   c. Delete EKSWorkloadClusterRole:
   ```bash
   aws iam detach-role-policy \
     --role-name EKSWorkloadClusterRole \
     --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
   
   aws iam delete-role --role-name EKSWorkloadClusterRole
   ```
   Status: ✅ Deleted
   
   d. Delete custom ACKEKSPolicy:
   ```bash
   aws iam delete-policy \
     --policy-arn arn:aws:iam::833542146025:policy/ACKEKSPolicy
   ```
   Status: ✅ Deleted

### Cleanup Notes

- **Deletion order is critical**: Workload cluster → Capability → Management cluster → IAM resources
- **Capability deletion**: Must wait for capability to fully delete before deleting management cluster
- **IAM cleanup**: Both inline and attached policies must be removed before deleting roles
- **Total cleanup time**: ~5-10 minutes for all resources
- **No manual VPC/subnet cleanup needed**: Clusters used existing VPC resources

## Cleanup Commands

**Note: All test resources have been cleaned up. These commands are provided for reference.**

```bash
# 1. Delete workload cluster
aws eks delete-cluster \
  --name eks-workload-cluster-test-only \
  --region us-west-2

# 2. Delete ACK capability (wait for workload cluster deletion to start)
aws eks delete-capability \
  --cluster-name eks-mgmt-cluster-ack-test-only \
  --region us-west-2 \
  --capability-name ack-test-capability

# 3. Delete management cluster (wait for capability deletion to complete)
aws eks delete-cluster \
  --name eks-mgmt-cluster-ack-test-only \
  --region us-west-2

# 4. Clean up IAM resources
# Detach and delete ACKCapabilityTestRole
aws iam detach-role-policy --role-name ACKCapabilityTestRole --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
aws iam detach-role-policy --role-name ACKCapabilityTestRole --policy-arn arn:aws:iam::833542146025:policy/ACKEKSPolicy
aws iam delete-role-policy --role-name ACKCapabilityTestRole --policy-name ACKEKSPolicy
aws iam delete-role --role-name ACKCapabilityTestRole

# Detach and delete EKSWorkloadClusterRole
aws iam detach-role-policy --role-name EKSWorkloadClusterRole --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
aws iam delete-role --role-name EKSWorkloadClusterRole

# Delete custom policy
aws iam delete-policy --policy-arn arn:aws:iam::833542146025:policy/ACKEKSPolicy
```

## Related Documentation

- [EKS Capabilities Documentation](https://docs.aws.amazon.com/eks/latest/userguide/eks-capabilities.html)
- [ACK Documentation](https://aws-controllers-k8s.github.io/community/)
- [EKS Pricing](https://aws.amazon.com/eks/pricing/)
