# Week 1: Cloud Operations Onboarding

## HarborTech Ticket Summary

Ticket ONB-2026-0001 required me to verify that my cloud training environment was ready before beginning client support work. I verified access to AWS Academy and Learner Lab, reviewed the Learner Lab operating restrictions, confirmed the permitted Region, reviewed IAM limitations, examined session and budget behavior, reviewed Reset behavior, located AWS documentation, and confirmed that my HarborTech Operations Playbook was ready for documentation.

## Client Impact

Environment readiness is important because an intern should not begin supporting a client without first confirming that required systems are accessible and understood. Access problems, permission restrictions, incorrect Regions, or misunderstood account limits could cause delays, incorrect troubleshooting, unnecessary costs, or changes outside the permitted environment.

Verifying the environment first allows the support technician to distinguish between an actual technical problem and an intentional platform restriction.

## AWS Services Involved

The Week 1 onboarding review involved the following AWS and cloud operations concepts:

* **AWS Academy** – Provides access to the course and training environment.
* **Learner Lab** – The controlled AWS sandbox used for course activities.
* **IAM** – Controls identities, roles, and permissions within AWS.
* **Region** – The geographic AWS location where cloud services operate.
* **AWS Account** – The controlled account where Learner Lab activities occur.
* **AWS Documentation** – Official technical documentation used to verify AWS behavior and procedures.
* **LabRole** – A preconfigured IAM role available within the Learner Lab.
* **LabInstanceProfile** – A provided instance profile that can be used by supported AWS resources when required.

## Virtualization Connection

Virtualization allows computing, storage, and networking resources to operate as software-defined services without the user directly managing the physical hardware underneath them.

Although the physical server is abstracted, cloud operations personnel still have important responsibilities. They must manage the correct AWS account, Region, permissions, security settings, costs, and platform restrictions.

Virtualization therefore reduces the need to interact directly with hardware, but it does not remove operational responsibility.

## Evidence Reviewed

During the Week 1 readiness review, I verified and reviewed the following evidence:

* AWS Academy access was available.
* Learner Lab access was available.
* The Learner Lab successfully started.
* The permitted Region was confirmed as **us-west-2**.
* The Learner Lab Readme was reviewed.
* IAM permissions were confirmed to be restricted.
* Creating an IAM user was not permitted in the sandbox.
* LabRole and LabInstanceProfile were identified as provided lab resources.
* Learner Lab access was confirmed to be session-based.
* Budget information can be delayed approximately 8 to 12 hours.
* Reset removes account resources and does not restore the budget.
* Official AWS documentation resources were located and reviewed.
* The HarborTech Operations Playbook repository was prepared for Week 1 documentation.
* No AWS resources were created during the Week 1 readiness review.

## Operational Analysis

The evidence shows that the training environment is functioning as expected and is ready for future HarborTech exercises.

A denied IAM user-creation attempt would not automatically indicate an AWS platform failure. The Learner Lab intentionally limits administrative permissions because it is a controlled sandbox. The IAM restriction is therefore an expected operating boundary.

The budget display should also not be considered real-time evidence because reporting can be delayed by approximately 8 to 12 hours. Resource cleanup should be performed when resources are no longer required rather than waiting for the displayed budget to change.

### Verified Findings

* AWS Academy is accessible.
* Learner Lab is accessible.
* The lab starts successfully.
* us-west-2 is available as the permitted Region.
* IAM contains intentional restrictions.
* Learner Lab provides LabRole and LabInstanceProfile.
* Budget reporting is delayed.
* Reset removes resources but does not restore the budget.
* AWS documentation is available for technical verification.

### Assumptions Avoided

I did not assume that an IAM permission denial represented a system failure. I compared the behavior with the documented Learner Lab restrictions before reaching a conclusion.

I also did not assume that the displayed budget represented current resource usage because the Learner Lab states that budget reporting may be delayed.

## Recommendation

The environment appears ready for Week 2 HarborTech support activities. AWS Academy and Learner Lab are accessible, the permitted Region has been identified, and the major operating boundaries have been reviewed.

Future work should continue to follow the Learner Lab Readme, use only permitted AWS services and permissions, clean up resources when they are no longer needed, and verify technical questions using official AWS documentation.

## Escalation Notes

No unresolved access or environment issues currently require instructor escalation.

If a future problem occurs that is not explained by the Learner Lab restrictions, I would document the error, service, Region, action attempted, and relevant non-sensitive evidence before escalating the issue to the instructor.

Credentials, passwords, access keys, tokens, and other sensitive information should never be placed in the public Operations Playbook.

## Lessons Learned

Week 1 demonstrated that cloud operations begins with verification rather than configuration. Before troubleshooting or changing an environment, an operations technician should understand what is accessible, what restrictions exist, and what evidence supports the conclusion.

I also learned that an error or denied action does not necessarily mean that the platform is broken. It may represent an intentional permission or sandbox boundary.

Operational decisions should be based on verified evidence rather than assumptions. Documentation is also important because it creates a record of what was checked, what was discovered, and what actions should occur next.

## Professional Vocabulary

**Virtualization:**
Technology that abstracts physical computing hardware and allows computing, storage, and networking resources to be provided through software.

**Evidence:**
Information or observations that can be verified and used to support a technical conclusion.

**Finding:**
A conclusion supported by reviewed evidence.

**Assumption:**
Something believed to be true without enough verified evidence.

**Escalation:**
Sending an unresolved problem to someone with additional authority, permissions, or technical expertise.

**Sandbox:**
A controlled environment where activities can be performed with specific limitations without providing unrestricted access to the full platform.

**Region:**
A geographic AWS location containing infrastructure where AWS services and resources can operate.

**IAM:**
AWS Identity and Access Management, which controls identities, roles, permissions, and access to AWS services.

**Operations Playbook:**
A documented collection of operational procedures, evidence, lessons learned, findings, and troubleshooting practices used to support consistent technical work.
