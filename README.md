# Multi-Department IAM & Governance Project

[![OCI](https://img.shields.io/badge/Oracle-Cloud-red?logo=oracle)](https://www.oracle.com/cloud/)
[![IAM](https://img.shields.io/badge/Focus-IAM%20%26%20Governance-blue)]()
[![License](https://img.shields.io/badge/License-MIT-green.svg)]()

> **OCI Architect Associate Certification Project**: Implementing enterprise-grade identity and access management with compartmental isolation, demonstrating real-world public sector cloud governance.

## 📋 Table of Contents
- [Project Overview](#-project-overview)
- [Architecture](#-architecture)
- [OCI Services Used](#️-oci-services-used)
- [Prerequisites](#-prerequisites)
- [Implementation Guide](#-implementation-guide)
- [Validation & Testing](#-validation--testing)
- [Cost Analysis](#-cost-analysis)
- [Security Considerations](#-security-considerations)
- [Key Learnings](#-key-learnings)
- [Troubleshooting](#-troubleshooting)
- [References](#-references)

---

## 🎯 Project Overview

### Objective
Design and implement a multi-tenant compartment structure with granular IAM policies, simulating a government organization with three independent departments: **Health**, **Education**, and **Infrastructure**. Each department operates with isolated resources, dedicated access controls, and independent budget tracking.

### GovTech Relevance
This architecture mirrors real-world government cloud deployments where:
- Departments require complete resource isolation
- Compliance mandates least-privilege access
- Budget accountability is critical for auditing
- Cross-department collaboration is controlled and logged

### What This Demonstrates
- Hierarchical compartment design
- Group-based access control (GBAC)
- Policy inheritance and scoping
- Dynamic groups for service-to-service authentication
- Tag-based cost allocation
- Budget monitoring and alerting
- Audit logging and compliance validation

---

## 🏗️ Architecture

### Compartment Hierarchy
```
Root Compartment (Tenancy)
│
└── GovTech-Parent
    │
    ├── Health-Dept
    │   ├── Health-Dev
    │   └── Health-Prod
    │
    ├── Education-Dept
    │   ├── Education-Dev
    │   └── Education-Prod
    │
    └── Infrastructure-Dept
        ├── Infrastructure-Dev
        └── Infrastructure-Prod
```

### Architecture Diagram
![Architecture Diagram](./diagrams/compartment_architecture.drawio.svg)

**Design Rationale:**
- **Parent Compartment**: Centralizes governance policies
- **Department Compartments**: Enforce organizational boundaries
- **Environment Sub-compartments**: Separate development and production workloads

### Resource Distribution
Each department contains:
- 🌐 **VCN** with public/private subnets
- 💻 **Compute Instance** (Oracle Linux)
- 🗄️ **Object Storage Bucket**
- 💾 **Block Volume**

---

## 🛠️ OCI Services Used

| Service | Purpose |
|---------|---------|
| **Compartments** | Logical isolation of resources |
| **IAM - Users & Groups** | Human identity management |
| **IAM - Dynamic Groups** | Service-to-service authentication |
| **IAM - Policies** | Fine-grained access control |
| **Tag Namespaces** | Metadata organization |
| **Tag Defaults** | Automatic resource tagging |
| **Budgets** | Cost monitoring and alerts |
| **Cost Analysis** | Tag-based spending reports |
| **Audit** | Access logging and compliance |
| **VCN** | Network isolation |
| **Compute** | Workload deployment |
| **Object Storage** | Data storage |
| **Block Volume** | Persistent storage |

---

## 📚 Prerequisites

### Required Access
- OCI Tenancy with administrator privileges
- OCI Free Tier account (sufficient for this project)

### Tools & Setup
```bash
# OCI CLI (optional but recommended)
brew install oci-cli  # macOS
# or
bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"

# Configure CLI
oci setup config

# Terraform (optional)
brew install terraform
```

### Knowledge Requirements
- Basic understanding of IAM concepts
- Familiarity with policy syntax
- OCI Console navigation

---

## 📋 Implementation Guide

### Phase 1: Compartment Setup

#### Step 1.1: Create Parent Compartment
```bash
# Using OCI Console
Identity & Security → Compartments → Create Compartment

Name: GovTech-Parent
Description: Parent compartment for all government departments
```

#### Step 1.2: Create Department Compartments
Create three compartments under `GovTech-Parent`:

| Name | Description | Parent |
|------|-------------|--------|
| Health-Dept | Health department resources | GovTech-Parent |
| Education-Dept | Education department resources | GovTech-Parent |
| Infrastructure-Dept | Infrastructure department resources | GovTech-Parent |

#### Step 1.3: Create Environment Sub-compartments
For **each** department, create:
- `{Department}-Dev` - Development environment
- `{Department}-Prod` - Production environment

![Screenshot](./screenshots/dept_compartments.png.jpg)

---

### Phase 2: IAM Groups & Users

#### Step 2.1: Create IAM Groups
Navigate to: `Identity & Security → Groups`

Create the following groups:

**Department-Specific Groups:**
- `HealthAdmins` - Full control over Health compartment
- `HealthDevelopers` - Limited to Health-Dev environment
- `HealthViewers` - Read-only access to Health resources
- `EducationAdmins`
- `EducationDevelopers`
- `EducationViewers`
- `InfrastructureAdmins`
- `InfrastructureDevelopers`
- `InfrastructureViewers`

**Cross-Department Groups:**
- `BudgetAnalysts` - View costs across all departments
- `SecurityAuditors` - Read-only access for compliance

#### Step 2.2: Create Test Users
Create at least 2 test users per group:

```
# Example naming convention
health-admin-1@example.com
health-dev-1@example.com
health-viewer-1@example.com
edu-admin-1@example.com
...
```

#### Step 2.3: Add Users to Groups
Assign each test user to their respective group. Example screenshot below for HealthAdmin users being added:

![Screenshot](./screenshots/health_admin_group.jpg)

---

### Phase 3: IAM Policies

#### Step 3.1: Department Admin Policies
**Policy Name**: `Health-Admins-Policy`  
**Compartment**: Applied at `Health-Dept` level

```
Allow group HealthAdmins to manage all-resources in compartment Health-Dept
```
![Screenshot](./screenshots/health_admins_policy.jpg)

Repeat for Education and Infrastructure departments.

![Screenshot](./screenshots/education_admins_policy.jpg)
![Screenshot](./screenshots/infrastructure_admins_policy.jpg)


#### Step 3.2: Developer Policies
**Policy Name**: `Health-Developers-Policy`  
**Compartment**: Applied at `HealthDept` level

```
Allow group HealthDevelopers to manage instance-family in compartment HealthDept
Allow group HealthDevelopers to use virtual-network-family in compartment HealthDept
Allow group HealthDevelopers to manage volume-family in compartment HealthDept
Allow group HealthDevelopers to manage object-family in compartment HealthDept
Allow group HealthDevelopers to inspect all-resources in compartment HealthDept
```
![Screenshot](./screenshots/health_developers_policy.jpg)

**Key Points:**
- Developers can **manage** resources in Dev
- Developers can only **inspect** (read-only) in Prod
- Uses resource families for granular control

#### Step 3.3: Viewer Policies
**Policy Name**: `Health-Viewers-Policy`

```
Allow group HealthViewers to inspect all-resources in compartment Health-Dept
Allow group HealthViewers to read all-resources in compartment Health-Dept
```

![Screenshot](./screenshots/health_viewers_policy.jpg)

#### Step 3.4: Cross-Department Policies
**Policy Name**: `Budget-Analysts-Policy`  
**Compartment**: Applied at `GovTech-Parent` level

```
Allow group BudgetAnalysts to read usage-reports in compartment GovTech-Parent
Allow group BudgetAnalysts to inspect compartments in compartment GovTech-Parent
Allow group BudgetAnalysts to read usage-budgets in compartment GovTech-Parent
```
![Screenshot](./screenshots/budget_analysts_policy.jpg)

**Policy Name**: `Security-Auditors-Policy`  
**Compartment**: Applied at `GovTech-Parent` level

```
Allow group SecurityAuditors to inspect all-resources in compartment GovTech-Parent
Allow group SecurityAuditors to read audit-events in compartment GovTech-Parent
Allow group SecurityAuditors to read policies in tenancy
```
![Screenshot](./screenshots/security_auditors_policy.jpg)

#### Step 3.5: Dynamic Groups
**Dynamic Group Name**: `Health-Compute-Instances`

**Matching Rule**:
```
All {instance.compartment.id = '<Health-Dept-OCID>'}
```
![Screenshot](./screenshots/health_compute_instances.jpg)

**Policy for Dynamic Group**:
```
Allow dynamic-group Health-Compute-Instances to read objects in compartment Health-Dept
Allow dynamic-group Health-Compute-Instances to manage object-family in compartment Health-Dept where target.bucket.name='health-app-data'
```

Repeat for other departments.

**All Policies Location**: `./policies/` directory

---

### Phase 4: Tagging Strategy

#### Step 4.1: Create Tag Namespace
Navigate to: `Governance → Tag Namespaces`

**Namespace Name**: `CostCenter`  
**Description**: Cost allocation and tracking tags

#### Step 4.2: Create Tag Keys
Create the following tags within `CostCenter` namespace:

| Tag Key | Description | Valid Values |
|---------|-------------|--------------|
| `Department` | Owning department | Health, Education, Infrastructure |
| `Environment` | Deployment environment | Dev, Prod |
| `Project` | Project identifier | Free text |
| `Owner` | Resource owner email | Free text |

#### Step 4.3: Set Tag Defaults
Configure automatic tagging for each compartment:

**Health-Dev Compartment**:
- `CostCenter.Department` = Health
- `CostCenter.Environment` = Dev

**Health-Prod Compartment**:
- `CostCenter.Department` = Health
- `CostCenter.Environment` = Prod

Repeat for all compartments.

**Screenshot Location**: `./screenshots/03-tag-namespace.png`

---

### Phase 5: Deploy Sample Resources

#### Deploy for Health Department

**5.1: Create VCN**
```
Name: health-vcn
CIDR Block: 10.0.0.0/16
Compartment: Health-Dev
Tags: 
  - CostCenter.Department: Health
  - CostCenter.Environment: Dev
  - CostCenter.Project: Health-Network
  - CostCenter.Owner: health-admin-1@example.com
```

**5.2: Create Compute Instance**
```
Name: health-app-server
Shape: VM.Standard.E2.1.Micro (Always Free)
Image: Oracle Linux 8
VCN: health-vcn
Subnet: Public Subnet
Compartment: Health-Dev
Tags: (same as above)
```

**5.3: Create Object Storage Bucket**
```
Name: health-app-data
Compartment: Health-Dev
Tags: (same as above)
```

**5.4: Create Block Volume**
```
Name: health-data-volume
Size: 50 GB
Compartment: Health-Dev
Tags: (same as above)
```

#### Repeat for Education and Infrastructure
Deploy the same resource types for:
- Education-Dev
- Infrastructure-Dev

**Screenshot Location**: `./screenshots/04-deployed-resources.png`

---

### Phase 6: Budget & Cost Tracking

#### Step 6.1: Create Department Budgets
Navigate to: `Governance → Budgets`

**Budget Configuration (Health Department)**:
```
Name: Health-Department-Budget
Compartment: Health-Dept
Target: $10.00 USD monthly
Alert Rule: 80% of budget
Alert Recipients: health-admin-1@example.com
```

Create similar budgets for Education ($10) and Infrastructure ($10).

#### Step 6.2: Generate Cost Report
Navigate to: `Governance → Cost Analysis`

**Create Custom Report**:
- Filter by: `Tag - CostCenter.Department`
- Group by: Department, Environment
- Date Range: Last 30 days

**Screenshot Location**: `./screenshots/05-cost-analysis.png`

---

## Validation & Testing

### Test Scenario 1: Positive Access Control
**Objective**: Verify admins can manage resources in their compartment

**Steps**:
1. Log in as `health-admin-1@example.com`
2. Navigate to Health-Dev compartment
3. Create a test compute instance
4. Verify instance creation succeeds
5. Delete the test instance

**Expected Result**: All operations succeed

**Screenshot Location**: `./validation/01-positive-admin-access.png`

---

### Test Scenario 2: Developer Environment Restrictions
**Objective**: Verify developers can only manage Dev, not Prod

**Steps**:
1. Log in as `health-dev-1@example.com`
2. Navigate to Health-Dev compartment
3. Create a test compute instance
4. Navigate to Health-Prod compartment
5. Attempt to create a compute instance

**Expected Result**: 
- Dev instance creation succeeds
- Prod instance creation fails with "Authorization failed"

**Screenshot Location**: `./validation/02-developer-restrictions.png`

---

### Test Scenario 3: Cross-Department Isolation
**Objective**: Verify department isolation

**Steps**:
1. Log in as `health-admin-1@example.com`
2. Attempt to view Education-Dept compartment
3. Attempt to create resources in Education-Dept

**Expected Result**: Both actions fail with authorization error

**Screenshot Location**: `./validation/03-cross-department-isolation.png`

---

### Test Scenario 4: Viewer Read-Only Access
**Objective**: Verify viewers cannot modify resources

**Steps**:
1. Log in as `health-viewer-1@example.com`
2. View Health-Dev compute instances
3. Attempt to stop an instance
4. Attempt to create a new instance

**Expected Result**: 
- Can view resources
- Cannot modify resources

**Screenshot Location**: `./validation/04-viewer-readonly.png`

---

### Test Scenario 5: Dynamic Group Access
**Objective**: Verify compute instances can access Object Storage via dynamic group

**Steps**:
1. SSH into `health-app-server`
2. Install OCI CLI on the instance
3. Configure instance principal authentication
4. Attempt to list objects in `health-app-data` bucket
5. Attempt to upload a test file

**Commands**:
```bash
# On the instance
oci os object list --bucket-name health-app-data --auth instance_principal
echo "test data" > test.txt
oci os object put --bucket-name health-app-data --file test.txt --auth instance_principal
```

**Expected Result**: Both operations succeed

**Screenshot Location**: `./validation/05-dynamic-group-access.png`

---

### Test Scenario 6: Budget Alert Validation
**Objective**: Verify budget alerts are triggered

**Steps**:
1. Create resources to approach budget threshold
2. Check for budget alert email
3. View budget status in console

**Expected Result**: Alert email received at 80% threshold

**Screenshot Location**: `./validation/06-budget-alert.png`

---

### Test Scenario 7: Audit Log Review
**Objective**: Verify all access attempts are logged

**Steps**:
1. Navigate to: `Observability → Audit`
2. Filter by:
   - Event Type: `com.oraclecloud.identitycontrolplane.createinstance`
   - Date: Last 7 days
3. Review both successful and failed attempts
4. Verify failed cross-department access is logged

**Screenshot Location**: `./validation/07-audit-logs.png`

---

## Cost Analysis

### Monthly Cost Breakdown (Estimated)

| Department | Dev Environment | Prod Environment | Total |
|------------|-----------------|------------------|-------|
| Health | $0.50 | $0.00 | $0.50 |
| Education | $0.50 | $0.00 | $0.50 |
| Infrastructure | $0.50 | $0.00 | $0.50 |
| **Total** | | | **$1.50/month** |

**Cost Optimization Notes**:
- Using Always Free tier compute shapes
- Object Storage charges minimal for small data
- No data transfer costs (internal traffic)
- Budgets set intentionally low for alerting demonstration

### Cost Report by Tag
**Group By**: `CostCenter.Department`

See detailed report: `./cost-analysis/monthly-cost-report.png`

---

##  Security Considerations

### Implemented Security Best Practices

1. **Least Privilege Access**
   - Users only have permissions necessary for their role
   - Developers cannot access production
   - Viewers have read-only access

2. **Compartment Isolation**
   - Complete resource separation between departments
   - Policy inheritance prevents privilege escalation

3. **Audit Logging**
   - All actions logged and retained
   - Failed access attempts tracked for security monitoring

4. **Service Authentication**
   - Dynamic groups used instead of API keys
   - Instance principals for secure service-to-service communication

5. **Network Isolation**
   - Each department has separate VCN
   - Private subnets for sensitive workloads
   - Security lists restrict traffic

### Potential Enhancements
- [ ] Implement MFA requirement for admin groups
- [ ] Add policy conditions for IP-based restrictions
- [ ] Enable Cloud Guard for threat detection
- [ ] Implement federation with identity provider
- [ ] Add DDoS protection policies

---

## 🎓 Key Learnings

### OCI Architect Associate Exam Alignment

This project directly addresses these exam domains:

**1. Identity and Access Management (15-20%)**
-  Compartment design and hierarchy
-  User, group, and policy management
-  Dynamic groups and instance principals
-  Policy syntax and inheritance

**2. Governance and Administration**
-  Tagging strategies
-  Cost tracking and budgets
-  Audit logging and compliance

**3. Networking**
-  VCN creation and configuration
-  Subnet design

**4. Compute**
- Instance deployment with IAM
-  Block volume management

### Technical Insights

**What Worked Well**:
- Tag defaults automated compliance
- Dynamic groups eliminated credential management
- Compartment hierarchy simplified policy management

**Challenges Faced**:
- Policy syntax can be tricky - test incrementally
- Tag propagation isn't instant - wait 1-2 minutes
- Dynamic group rule testing requires patience

**Best Practices Discovered**:
1. Always create policies at the compartment level first
2. Use descriptive naming conventions for all resources
3. Document OCIDs for compartments immediately
4. Test negative scenarios as thoroughly as positive ones
5. Budget alerts should be set lower than you think

---

## 🔧 Troubleshooting

### Common Issues

**Issue 1: "Authorization failed" when creating resources**
- **Cause**: Policy not applied or incorrect compartment selected
- **Solution**: Verify policy syntax, wait 30 seconds for propagation, check user's group membership

**Issue 2: Dynamic group not working**
- **Cause**: Matching rule incorrect or policy missing
- **Solution**: Verify instance compartment OCID, ensure instance has been restarted after dynamic group creation

**Issue 3: Tags not appearing on resources**
- **Cause**: Tag defaults not applied or resource created before tag default
- **Solution**: Manually apply tags, or recreate resource after tag default setup

**Issue 4: Budget alerts not received**
- **Cause**: Email not verified or budget threshold not crossed
- **Solution**: Check spam folder, verify email in notifications, lower budget threshold for testing

**Issue 5: Cannot view cost reports**
- **Cause**: Insufficient permissions
- **Solution**: Ensure user has `read usage-reports` permission at appropriate compartment level

---

## 📚 References

### Official OCI Documentation
- [Compartments Overview](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingcompartments.htm)
- [IAM Policy Reference](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/policyreference.htm)
- [Dynamic Groups](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingdynamicgroups.htm)
- [Tagging Resources](https://docs.oracle.com/en-us/iaas/Content/Tagging/Concepts/taggingoverview.htm)
- [Cost Management](https://docs.oracle.com/en-us/iaas/Content/Billing/Concepts/billingoverview.htm)

### Learning Resources
- [OCI Architect Associate Certification](https://education.oracle.com/oracle-cloud-infrastructure-2024-architect-associate/pexam_1Z0-1072-24)
- [OCI Free Tier](https://www.oracle.com/cloud/free/)
- [OCI CLI Documentation](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cliconcepts.htm)


---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**Note**: This is a demonstration project for certification preparation. Always follow your organization's security policies and compliance requirements in production environments.

