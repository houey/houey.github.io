# AWS Organizations Migration: Lessons from a large digital native financial services company's journey. 

*A real world guide to migrating from a workload bearing Organization Management Account to a dedicated security focused, isolated AWS Organization Management Account*

If you're reading this, chances are you're staring at the same uncomfortable reality that countless AWS customers face: your Organization Management Account is also running production workloads. You're not alone, and more importantly, you're not stuck with this architecture forever.

## The Problem We All Face

Back in 2018, when many of us were first setting up AWS Organizations, the implications weren't as clear. The security best practices hadn't been as widely evangelized, and frankly, AWS hadn't made the risks as obvious as they are today. Fast forward to 2024, and we're living with the consequences of those early architectural decisions.

The issue? Having your Organization Management Account (formerly the "Master" account) also serve as a production workload account creates a perfect storm of security vulnerabilities:

- **Privilege escalation risks** become exponentially more dangerous
- **Account to account lateral movement** becomes trivial for attackers
- **Compliance frameworks** become nearly impossible to satisfy
- **Blast radius** of any compromise extends to your entire AWS footprint

The situation becomes even more precarious if you've enabled powerful services like CloudFormation StackSets, Control Tower, or other organization-wide tools in the same account where your critical workloads and data reside.

## Why AWS Won't Save Us (Yet)

Here's the frustrating truth: AWS knows this is a problem. Customers have been requesting a "friendly path" to rehome the Organization Management Account for years. Unfortunately, it's still not prioritized on their roadmap. This leaves us with one option: the nerve-wracking process of migrating to an entirely new AWS Organization.

**Pro tip**: If you have access to AWS Support, file tickets and add your voice to the enhancement request. The more customer pressure, the more likely AWS will prioritize this feature.

## Our Migration Journey

At the tail end of 2022, we undertook this migration for a well-known digital-native enterprise. Here's what we learned, what worked, and what nearly gave us heart attacks along the way.

### The Pre-Migration Reality Check

Before diving into our approach, let's acknowledge what made our situation manageable:

- **No Identity Center** adoption yet (this would have complicated everything)
- **Minimal RAM/VPC sharing** across the organization
- **Limited use of PrincipalOrgId/ResourceOrgId** in policies

If your environment is more complex, plan accordingly and consider engaging AWS Professional Services.

## The Master Plan

### Phase 1: Foundation Building

**1. Create the New Organization Structure**
- Set up a dedicated, clean Organization Management Account
- Establish a new CloudTrail OrgTrail for comprehensive logging
- Design your OU structure (keep it flat, but leave room for future acquisitions)

**2. Implement Baseline Security**
- Deploy foundational Service Control Policies (SCPs)
- Focus on the "easy wins": defang root access, prevent CloudTrail deletion, restrict problematic instance types, limit IAM user creation
- Set up the new OrgTrail to deliver logs to your existing delegated Log Archive bucket.

### Phase 2: Coordination and Preparation

**3. Stakeholder Alignment**
- Work with AWS to ensure any enterprise specific customizations are carried forward
- Coordinate with Finance/CostOps teams for billing continuity
- **Critical warning**: You WILL lose native billing history when the old organization dies. Plan accordingly.

**4. Technical Preparation**
- Update the OrganizationAccountAccessRole in all member accounts to be assumable from the new Organization Management Account
- Update any principalOrgId or resourceOrgId references in existing policies to accept both old and new organization IDs
- If you've shared AMIs by organization, account for this in your migration plan and don't forget to update the sharing settings to the new Organization ID

### Phase 3: The Migration Dance

**5. Account Migration Strategy**
- Note that any inflight *Suspended* AWS account that are going through their 90 day deletion cycle will hold up this process.  Wait or remove them from the org before suspendeding if needed.
- Start with smaller, less business critical accounts
- Move most remaining accounts to the new organization
- Save the legacy Organization Management Account for last (you have no choice, because you have to delete the legacy AWS Organization)

**6. Special Considerations**
- **RAM shared resources**: Move accounts together where possible to minimize disruption
- **CloudFormation StackSets**: Consider creating a staging OU in the old organization to strip StackSets before migration, then reapply them in the new organization
- **Payment information**: Be prepared to add temporary payment methods to accounts during the transition

## The Hard-Won Lessons

### Control Tower Complexity

If you're using Control Tower, seriously consider decommissioning the landing zone before migration. We did this and never turned it back on. Control Tower adds significant complexity to the migration process, and the benefits often don't justify the additional risk and effort.

### The StackSets Challenge

We chose to mirror our CloudFormation StackSets in the new organization rather than attempting to migrate them. Here's why:

1. **Cleaner slate**: Starting fresh eliminated potential configuration drift
2. **Staging approach**: We created a staging OU in the old organization to strip StackSets from accounts before migration
3. **Controlled reapplication**: StackSets were reapplied automatically when accounts landed in their new OUs

**Caution**: If you depend on crossaccount roles generated by StackSets in various policies, this approach can be breaking. The IAM identifiers will change, requiring policy updates.

### Timing Is Everything

The best time to migrate your organization structure is always yesterday. If you're starting down the path of VPC Endpoints, RAM, or comprehensive Service Control Policies, it's nearly impossible to maintain proper security posture while managing the organization from a workload-bearing account.

## Critical Questions to Consider

Before you begin, honestly assess these factors:

- **Pricing and discounts**: How will you guarantee your complete pricing regime survives the migration?
- **Timing risks**: Are there potential issues when accounts are briefly "standalone" during the transition?
- **Custom quotas**: What organizational or organization wide API rate limits, resource quotas, or custom configurations could be lost?
- **Identity Center**: If you're using it, engage AWS support before attempting migration
- **Compliance requirements**: How will you maintain audit trails and compliance during the transition?

## The Bottom Line

Migrating AWS Organizations is not for the faint of heart, but it's often necessary for maintaining a secure and compliant cloud environment. The process requires careful planning, stakeholder coordination, and a healthy respect for what can go wrong.

The good news? It's absolutely doable, and the security benefits are worth the effort. Just remember: plan extensively, test thoroughly, and work closely with AWS throughout the process.

---

*Have you been through an AWS Organizations migration? What lessons did you learn? Share your experiences and help the community navigate this challenging but necessary process.*
