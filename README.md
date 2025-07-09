 Lab 7.1: Remediating an Incident by Using AWS Config and Lambda 
Summary and Walkthrough
Overview & Objectives
Goal: Use AWS Config to detect unauthorized changes to EC2 security-group inbound rules and automatically remediate them via a Lambda function.

Key Services: IAM, AWS Config, Amazon VPC Security Groups, AWS Lambda, CloudWatch Logs.

Outcome: Any change that opens ports beyond HTTP/HTTPS is automatically closed again.

Task 1: Examining & Updating IAM Roles
Inspect the AwsConfigLambdaSGRole inline policy.

What it does: Grants the Lambda function rights to describe and modify EC2 security-group rules and write CloudWatch logs.

Attach the AWS-managed AWS_ConfigRole policy to the AwsConfigRole role.

Why: Allows AWS Config to record configuration snapshots to S3 and read related resource metadata.

Troubleshooting: If you tried editing the inline JSON but saw an Access-Analyzer error, back out to the Roles → AwsConfigRole page and use Add permissions → Attach policies instead.

Task 2: Setting Up AWS Config
In AWS Config “Settings”:

Enable recording for Specific resource types → AWS EC2 SecurityGroup at Continuous frequency.

Select the AwsConfigRole you prepared.

Verify: Wait for the “Recording is on” banner.

Inventory: On the Resources page, filter by Network & security → AWS::EC2::SecurityGroup.

Common hiccup: If no resources appear, collapse the Aggregators menu, ensure you’re in the right region (us-west-2), and refresh after a few minutes.

Task 3: Simulating a Security Incident
In the VPC console, filter to Lab VPC → Security groups → select LabSG1.

Edit inbound rules: change HTTP to 0.0.0.0/0, then add HTTPS (443), SMTPS (465), IMAPS (993) all from Anywhere-IPv4.

Capture a screenshot showing all four rules.

You’re emulating an accidental or malicious opening of SMTPS/IMAPS that you don’t want.

Task 4: Creating a Custom AWS Config Rule
Copy the pre-provisioned Lambda ARN from the AWS Details panel.

In AWS Config → Rules, click Add rule → Create custom Lambda rule:

Function ARN: your copied ARN

Name: EC2SecurityGroup

Description: “Restrict inbound ports to HTTP and HTTPS”

Trigger: When configuration changes on AWS EC2 SecurityGroup

Parameter: debug = true

Save and wait (~2–3 minutes) for the initial “Last successful evaluation” to show Compliant with annotation “Permissions were modified.”

Tip: Hit the refresh icon if it stays on “Not available.”

Task 5: Verifying Remediation & Reviewing Code
Revisit LabSG1 in the VPC console:

Inbound rules now show only HTTP (80) and HTTPS (443), for both IPv4 and IPv6. SMTPS/IMAPS are gone.

In Lambda → awsconfig_lambda_security_group → Code:

Line 2: import boto3 (plus botocore, json).

Line 9: REQUIRED_PERMISSIONS array lists only port 80 and 443 rules (IPv4 & IPv6).

Line 117: describe_security_groups() fetches current rules.

Line 129: if debug: block to emit extra log output.

No edits required — the function already enforces exactly your desired state.

Task 6: Checking CloudWatch Logs
In CloudWatch → Logs → Log groups, click /aws/lambda/awsconfig_lambda_security_group.

Choose Search all log streams, filter for “revoking for”.

Expand a log event that shows lines like:

rust
Copy code
Revoking for sg-0ebfc0f7711197e63: port 465
Revoking for sg-0ebfc0f7711197e63: port 993
Screenshot one of these entries — proof of auto-remediation.

Scope & Key Takeaways
Scope: Tracks only EC2 Security Groups in a single region, but could be extended to other resource types (RDS, IAM, etc.) or additional unwanted ports.

Significance: Demonstrates fully automated detection + remediation with no human intervention — critical for rapid incident response.

Lessons Learned:

How AWS Config’s detective (configuration-change) mode can tie into corrective (Lambda) actions.

Importance of least-privilege IAM roles for both Config and Lambda.

How CloudWatch Logs serve as your audit trail for automated security enforcement.

Pushing Your Work to GitHub
Create a new repository

Go to github.com → click New (top-left) → New repository.

Owner: your username (e.g. ime-cloud-sec-analyst)

Repository name: something like aws-config-sg-remediation

Description (optional): “AWS Config + Lambda auto-remediation for EC2 security groups”

Visibility: Public or Private as you prefer.

(Optionally) check Add a README and .gitignore (Python).

Click Create repository.

Push your local files

bash
Copy code
git init
git add awsconfig_lambda_security_group.py README.md
git commit -m "Initial lab: AWS Config + Lambda SG remediation"
git branch -M main
git remote add origin https://github.com/<your-user>/aws-config-sg-remediation.git
git push -u origin main
Document your steps

README.md, summarize the lab’s tasks (Tasks 1–6), include screenshots, and key lessons learned.

Congratulations—you’ve built, tested, and audited a fully automated security remediation pipeline in AWS! 🚀
