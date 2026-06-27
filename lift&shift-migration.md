# Lift-and-Shift (Rehost) Migration Framework

Here is the complete, end-to-end **Lift-and-Shift (Rehost) Migration Framework** for your Notion workspace, combining your high-level lifecycle steps, detailed task plans (including multi-cloud AWS-to-GCP configurations), and the foundational Landing Zone updates.

**🚀 Lift-and-Shift Migration Framework**
- **Framework Type:** Cloud-to-Cloud / On-Prem-to-Cloud Rehost
- **Primary Objective:** Standardize discovery, execution, and cutover of mass virtual machine and database migrations.
- **Target Environment:** Multi-Cloud / Hybrid (e.g., AWS to GCP Sole Tenant/Standard Compute)

## Phase 1: Pre-Migration & Readiness Assessment

Focuses on establishing the "Why", defining success, and setting up the administrative baseline.

### 1.1 Business Alignment & Scope
- Define **Migration Success Criteria**: Understand exactly why the customer is moving (e.g., data center exit, cost savings, multi-cloud strategy). This defines the true success criteria for the migration.

## Phase 2: Discovery & Architecture Design

Focuses on inventory mapping, dependency tracking, and drawing the cloud blueprint.

### 2.1 Inventory & Cost Approximation
- **Total Server & OS Inventory**: Extract total server counts, OS distributions, resource allocations (vCPU/RAM), and storage configurations.
  - 💡 Architect Note: OS inventory directly dictates your replication tool compatibility (e.g., Google Migrate for Compute / AWS Application Migration Service).
- **Cost Approximation Sheet**:
  1. Calculate post-migration compute/storage costs.
  2. Include network egress/traffic costs incurred during continuous data replication.
  3. Map out software licensing costs (Migration tools, third-party network appliances like Palo Alto).
  4. Plan properly when Reservations (RI / Committed Use Discounts) will be purchased and applied.

### 2.2 Application Dependency Mapping
- **Service Mapping**: Run dependency discovery tools (e.g., Stratozone or automated discovery agents) if application mappings aren't natively documented.
- **First Draft Dependency Graph**: Identify which servers communicate with each other to establish logical **Move Groups** (preventing split-brain applications or high-latency issues across environments during migration).

### 2.3 Network Architecture & High-Level Design (HLD)
- **Detailed Network Design**: Finalize VPN/Interconnect configurations, IP allocations (ensuring zero overlapping CIDR blocks), and routing paths.
- **Security Baseline Evaluation**: Compare source security postures against destination rules (e.g., mapping AWS Security Groups to GCP Firewall Rules, aligning AWS Palo Alto configurations to GCP Palo Alto routes).
- **High-Level Design (HLD)**: Draw and secure approval for the target state cloud architecture.

## Phase 3: SOW and Project Kickoff
- **Statement of Work (SOW) & Kickoff**: Finalize the SOW, prepare the kick-off presentation deck, and align stakeholders at a formal project kickoff meeting.
- **Team & Identity Provisioning**: Collect team details and submit forms for platform-level access (e.g., IAM roles for AWS/GCP, project team channels).

## Phase 4: Environment Setup & Replication

Setting up the cloud foundation, configuring the migration engines, and beginning the data copy phase.

- **Project Governance**: Create the master Project Plan & Delivery Schedule to manage dependencies and track milestones.

### 4.1 Landing Zone Setup & Core Foundation
Before any workload is migrated, the destination environment must be structured, secured, and governed according to enterprise baselines.

#### Resource Hierarchy & Account/Project Structure
- Define and provision the organizational structure (e.g., AWS Organizations/OUs or GCP Folders).
- Separate environments cleanly by creating dedicated projects/accounts for Management/Security, Shared Services (AD/DNS/Network/Firewalls), Dev/Test, and Production.

#### Identity & Access Management (IAM)
- Integrate the destination cloud with the customer’s Identity Provider (IdP) via SAML/OIDC (e.g., Okta, Entra ID).
- Implement **Role-Based Access Control (RBAC)** to ensure the migration team, security team, and operations team have strict least privilege access.

#### Security Guardrails & Compliance Baselines
- Enable organization-level policies (e.g., AWS SCPs or GCP Organization Policies) to restrict unapproved regions, block public storage buckets, and enforce encryption.
- Turn on native security monitoring tools (e.g., AWS Security Hub, GCP Security Command Center, and CloudTrail/Cloud Logging).

#### Core Network Topology (The Landing Zone Network)
- Deploy the core network infrastructure (VPCs, Subnets, and Cloud Routers/Transit Gateways).
- **Inter-Cloud Connectivity**: Establish and test the dedicated site-to-site connectivity (VPN or Interconnect) between the source (AWS) and destination (GCP).
- **Network Appliances Deployment**: Provision and configure centralized security perimeters, such as deploying Palo Alto firewalls to inspect cross-cloud traffic.

### 4.2 Core Shared Services Provisioning
- **Shared Infrastructure**: Deploy baseline operational infrastructure required by all workloads (e.g., Active Directory (AD) servers within the Shared Services project).
- **DNS Infrastructure & Routing Baseline**: Configure AD DNS servers, establish DNS forwarders, and set up cross-project DNS peering wherever required or use cloud DNS (e.g., peering Dev/Prod projects to the Shared Services DNS).
- **Landing Zone Verification**: Deploy a simple test VM in the new environment to perform a thorough ping-and-routing sweep to ensure it successfully resolves DNS and communicates across the VPN/Interconnect.

### 4.3 Creation of Move Groups
- Based on application dependency discovery tools run in 2.2, create move groups of servers which will go together.
- Get created move groups validated and signed off from application owners and architects.

### 4.4 Migration Engine & Target Compute Configuration
- **Migration Appliance Deployment**: Configure the migration engine (e.g., Google Migrate for Compute / AWS Application Migration Service) within the landing zone and establish secure replication tunnels.
- **Target Architecture Provisioning**: Pre-configure specialized target compute infrastructure to accommodate specific OS or database licensing compliance.
- Spin up target **Load Balancers (LBs)** in a dormant state, ready to receive backend instances during cutover.

## Phase 5: Execution, Cutover & Testing

The operational playbook for replicating, testing, and cutting over each Move Group.

For every designated Move Group, execute the following sprint loop:
1. Continuous Replication
2. Test Clone / Test Cutover
3. Infra Sanity & App Validation
4. Production Cutover (Final Delta Sync)
5. DNS Switch & License Activation

### 5.1 Step-by-Step Move Group Playbook
- **Continuous Replication**: Begin background block-level data synchronization of the move group's servers (handling both stopped state and active state groups sequentially).
- **Test Clone / Test Cutover**: Launch a non-disruptive clone of the workloads in an isolated network environment to verify standard booting behavior.
- **Infra Sanity & Application Testing**: Validate that the application boots, registers with local DNS, and can talk to its database or dependent services. Resolve any migration tool errors or OS-level hitches here.
- **Production Cutover**: Schedule a maintenance window, stop traffic to the source, execute the final delta sync, and bring the new servers live.
- **DNS/Network Transition**: Update Cloud DNS zones, registers, and forwarders to point traffic to the new instances or newly created load balancers.
- **License Activation**: Activate OS or third-party software licenses on the newly migrated infrastructure.

## Phase 6: Hypercare, Day-2 Operations & Sign-off

Handing over the keys to the enterprise operations team.

### 6.1 Monitoring & Observability
- **Cloud Ops Dashboards**: Set up centralized logging, infrastructure performance dashboards, and automated alert rules for CPU, memory, and disk space.
- **Service Level Troubleshooting**: Maintain a high-alert triage loop to address performance bottlenecks, routing hitches, or application configuration errors encountered immediately post-cutover.

### 6.2 Knowledge Transfer & Offboarding
- **Console Walkthrough & KT**: Conduct formal knowledge transfer sessions with the customer’s internal cloud operations team covering the new environment layout, access paths, and deployment rules.
- **Hypercare Window**: Provide dedicated, close-knit engineering support for an agreed period (e.g., 1–2 weeks post-migration).
- **Project Sign-off**: Secure formal stakeholder sign-off to officially close out the migration project successfully.

