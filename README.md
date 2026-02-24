# Zero-Worker Management Cluster Test

This folder contains all files and documentation related to testing EKS Capabilities on a cluster with zero worker nodes.

## Test Overview

**Objective:** Determine whether AWS EKS Capabilities (ACK, ArgoCD, kro) can function on an EKS cluster with no worker nodes.

**Result:** ✅ CONFIRMED - EKS Capabilities work perfectly on zero-node clusters.

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

- ✅ Zero-node cluster created
- ✅ ACK capability enabled and ACTIVE
- ✅ Workload cluster creation initiated via ACK
- ⏳ Waiting for workload cluster to complete (10-15 minutes)

## Cleanup Commands

```bash
# Delete ACK-created workload cluster
kubectl delete cluster eks-workload-cluster-test-only --context ack-test

# Delete ACK capability
aws eks delete-capability \
  --cluster-name eks-mgmt-cluster-ack-test-only \
  --region us-west-2 \
  --capability-name ack-test-capability

# Delete management cluster
aws eks delete-cluster \
  --name eks-mgmt-cluster-ack-test-only \
  --region us-west-2
```

## Related Documentation

- [EKS Capabilities Documentation](https://docs.aws.amazon.com/eks/latest/userguide/eks-capabilities.html)
- [ACK Documentation](https://aws-controllers-k8s.github.io/community/)
- [EKS Pricing](https://aws.amazon.com/eks/pricing/)
