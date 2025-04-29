# AWS_DevOps_Cost_Optimization


Objective:

Automatically identify and clean up unused AWS resources like:
	•	Unattached EBS volumes
	•	Old AMIs
	•	Unassociated Elastic IPs
	•	Orphaned snapshots
	•	Idle Load Balancers

⸻

AWS Resource Janitor – Step-by-Step Implementation

⸻

Step 1: Create an AWS Config Rule
	•	Navigate to: AWS Config → Rules → Add rule
	•	Use managed rules like:
	•	ec2-volume-inuse-check (for EBS)
	•	ec2-unused-address-check (for Elastic IPs)
	•	Custom Lambda-backed rule for old AMIs or idle ELBs

These rules will continuously scan and log unused resources in Config.

⸻

Step 2: Create a Lambda Function to Evaluate Resources
	•	Use Boto3 to:
	•	Describe EC2 volumes, snapshots, images, EIPs, ELBs
	•	Filter unused ones (e.g., State=available, AttachmentSet=[])
	•	Check for custom tags like "Keep":"True" to exclude sensitive resources
	•	Log findings in CloudWatch

Trigger:
Set the Lambda to run every 24 hours using EventBridge Scheduler (cron).

⸻

Step 3: Add an Approval Layer via SNS
	•	If a resource qualifies for cleanup:
	•	Publish its details to an SNS topic with a message like:

Unused EBS volume detected:
Volume ID: vol-0123abc
Size: 100 GB
Age: 45 days

Subscribe your email or Slack via Lambda integration or webhook to approve before deletion (optional step if you want human approval).

⸻

Step 4: Create a Second Lambda to Perform Cleanup
	•	This Lambda checks:
	•	If the resource is still unused
	•	If a TTL (time-to-live) tag is expired
	•	Or if approval was granted
	•	If true, it runs:
	•	delete_volume() for EBS
	•	deregister_image() and delete_snapshot() for AMIs
	•	release_address() for EIPs
	•	delete_load_balancer() for ELB

Ensure proper IAM roles and tagging strategy for safety.

⸻

Step 5: Use AWS Systems Manager (SSM) for Safe Execution (Optional)
	•	Create an SSM Automation Document for each resource type deletion
	•	Add guardrails like:
	•	Notify team before delete
	•	Backup snapshot creation
	•	Approval with automation:ExecuteScript

⸻

Step 6: Track Everything in CloudTrail + Log to S3
	•	Enable logging of:
	•	Lambda executions
	•	Delete actions
	•	Approvals
	•	Store them in a centralized S3 bucket with versioning enabled
	•	Add Athena/QuickSight for audit reports (optional)

⸻

Step 7: Slack or Email Notification of Cleanups
	•	Use SNS or Lambda → API Gateway → Slack Webhook
	•	Notify:
✅ Cleaned up 2 unattached EBS volumes (200 GB saved)
✅ Deleted 3 unused AMIs older than 60 days

✅ Cleaned up 2 unattached EBS volumes (200 GB saved)
✅ Deleted 3 unused AMIs older than 60 days

Step 8: (Bonus) Add Budget Alerts for Visual Validation
	•	Use AWS Budgets to validate that savings are being realized
	•	Trigger alerts if usage drops or shoots unexpectedly

⸻

Tools Used:
	•	AWS Config
	•	Lambda (Python + Boto3)
	•	EventBridge Scheduler
	•	SNS
	•	SSM Automation
	•	CloudTrail
	•	Slack (optional)
	•	Tags like AutoDelete, Keep, TTL, Environment

⸻

End Result:
	•	Fully automated cloud hygiene
	•	Consistent savings
	•	Clean, secure, and cost-optimized environment
	•	Audit-compliant logs with human approval options
