🌐 terraform-aws-http-server
Deploy an Apache HTTP web server on AWS EC2 using Terraform — automated from infrastructure provisioning to web page serving, using the AWS default VPC for simplicity and speed.

Terraform AWS Apache Amazon Linux Region License

📸 Result
Once deployed, Terraform outputs a ready-to-use URL:

application_url = "http://X.X.X.X"
🏗️ Architecture

**Internet -----------> Browser ----------------> │ Security Group│ -------------------> | AWS Default VPC | ----------->│ EC2 Instance │-------> | Apache httpd**

✨ Features
✅ Default VPC — No custom VPC needed; uses your existing AWS default VPC
✅ Dynamic Security Group — Ingress rules built from a locals list, easy to extend
✅ Latest AMI auto-resolved — Always fetches the most recent Amazon Linux 2023 x86_64 AMI
✅ Automated web server — Apache installed and configured via EC2 user data; zero manual steps
✅ SSH locked to your IP — Port 22 restricted to your CIDR via my_ip_cidr variable
✅ Environment tagging — All resources tagged with Environment, Project, ManagedBy
✅ Configurable — Region, instance type, key pair, and environment controlled via variables.tf
📁 Project Structure
terraform-aws-http-server/
├── main.tf            # Provider, data sources, security group, EC2 instance
├── variables.tf       # Input variable declarations
├── terraform.tfvars   # Your variable values (key_name, my_ip_cidr)
├── outputs.tf         # Instance ID, Public IP, DNS, Application URL
└── userdata.sh        # Apache install + custom HTML page bootstrap script
🛠️ Prerequisites
Tool	Version	Reference
Terraform	≥ 1.5.0	Install Guide
AWS CLI	≥ 2.x	Install Guide
AWS Account	—	With EC2 + VPC read permissions
EC2 Key Pair	Existing	Created in AWS Console → EC2 → Key Pairs
Minimum IAM Permissions
{
  "Effect": "Allow",
  "Action": [
    "ec2:*",
    "vpc:Describe*"
  ],
  "Resource": "*"
}
Configure AWS credentials
aws configure
# AWS Access Key ID:     <your-key>
# AWS Secret Access Key: <your-secret>
# Default region name:   ap-south-1
# Default output format: json
🚀 Quick Start
1. Clone the repository
git clone https://github.com/adasgupt-86//terraform-ec2-apache-http-deployment.git
cd terraform-aws-http-server
2. Update terraform.tfvars
key_name   = "your-aws-key-pair-name"   # Must already exist in ap-south-1
my_ip_cidr = "YOUR.PUBLIC.IP/32"        # Find it: curl ifconfig.me
💡 Find your public IP quickly: curl -s ifconfig.me && echo "/32"

3. Initialise Terraform
terraform init
4. Preview the plan
terraform plan
5. Deploy
terraform apply
Type yes when prompted. Deployment completes in ~2 minutes.

6. Open the application
Terraform prints this at the end:

application_url = "http://X.X.X.X"
⚙️ Variables
Variable	Type	Default	Required	Description
aws_region	string	ap-south-1	No	AWS region to deploy into
instance_type	string	t3.micro	No	EC2 instance type
environment	string	dev	No	Environment tag value
key_name	string	—	Yes	Existing EC2 key pair name
my_ip_cidr	string	—	Yes	Your IP in CIDR notation for SSH (e.g. 1.2.3.4/32)
Deploying to a different region
# terraform.tfvars
aws_region = "us-east-1"
key_name   = "my-us-east-key"
my_ip_cidr = "203.0.113.10/32"
📤 Outputs
Output	Description	Example
instance_id	EC2 instance ID	i-0abc1234def56789
public_ip	Public IPv4 address	13.235.X.X
public_dns	AWS public DNS hostname	ec2-13-235-X-X.ap-south-1.compute.amazonaws.com
application_url	Clickable HTTP URL	http://13.235.X.X
🔒 Security Notes
Consideration	Implementation
SSH access	Restricted to my_ip_cidr only — never open to 0.0.0.0/0
HTTP access	Open to all (0.0.0.0/0) — required for a public web server
Key pair	Uses a pre-existing AWS key pair; private key stays on your machine
Outbound traffic	All outbound allowed (required for dnf updates in userdata)
⚠️ This setup serves over HTTP (unencrypted). For production workloads, add HTTPS via AWS ACM + Application Load Balancer, or mod_ssl with Let's Encrypt.

📌 Resources Created
Resource	Name	Notes
aws_security_group	web-sg	HTTP + SSH ingress, dynamic rules
aws_instance	http-web-server	Amazon Linux 2023, t3.micro
Data sources (aws_vpc, aws_subnets, aws_ami) are read-only — they reference existing AWS resources and create nothing new.

🐛 Troubleshooting
Symptom	Likely Cause	Fix
Page not loading in browser	SG missing port 80	Check inbound rules in AWS Console
terraform apply fails on AMI	Wrong region or filter	Confirm aws_region matches your key pair's region
SSH connection times out	Wrong my_ip_cidr	Update with current IP: curl ifconfig.me
Apache not running	userdata.sh error	SSH in → sudo cat /var/log/cloud-init-output.log
key_name not found error	Key doesn't exist in region	Create it in AWS Console → EC2 → Key Pairs
SSH in and check Apache
ssh -i ~/.ssh/your-key.pem ec2-user@<public_ip>

**Check Apache service status**
sudo systemctl status httpd

**View userdata execution logs**
sudo cat /var/log/cloud-init-output.log | tail -40

**Test Apache responds locally**
curl -I http://localhost

🧹 Teardown
Destroy all created resources to avoid AWS charges:

terraform destroy
Type yes to confirm. This removes the EC2 instance and security group.

The default VPC is not affected — it pre-existed and is not managed by this configuration.

📜 License
This project is licensed under the MIT License.

🙌 Author
Built with ❤️ using Terraform · AWS EC2 · Apache HTTP Server



# Deploy Grafana for observability though Cloudwatch on this and all other EC2 instances in this AWS account 

📊 **Grafana on AWS EC2 with CloudWatch Dashboards**

This project demonstrates how to deploy Grafana on an Amazon Linux 2023 EC2 instance and configure CloudWatch as a data source to build multiple monitoring dashboards for AWS infrastructure.

🚀 **Architecture**
User Browser
    │
    ▼
EC2 Instance (Amazon Linux 2023)
    │
    ▼
Grafana Server (Port 3000)
    │
    ▼
AWS CloudWatch Metrics

🧰 **Prerequisites**
AWS Account 
EC2 instance (t3.micro / t3.micro)
Key pair for SSH access
Security Group configured:
SSH (22)
Grafana (3000)

🖥️ **EC2 Setup via terraform**

Launch EC2 instance through terraform as describe above

AMI: Amazon Linux 2023
Instance type: t3.micro (Free Tier)
Storage: 8–10 GB
Open ports:
22 (SSH)
3000 (Grafana)
📦 Install Grafana

SSH into EC2:

ssh -i key.pem ec2-user@<EC2_PUBLIC_IP>

**Install Grafana:**

sudo dnf update -y

sudo tee /etc/yum.repos.d/grafana.repo > /dev/null <<EOF
[grafana]
name=Grafana Repository
baseurl=https://rpm.grafana.com
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
EOF

sudo dnf install grafana -y

Start service:

sudo systemctl enable grafana-server
sudo systemctl start grafana-server

Check status:

sudo systemctl status grafana-server

🌐 **Access Grafana**

Open browser:

http://<EC2_PUBLIC_IP>:3000

Login:

Username: admin
Password: admin (change on first login)
🔐 **IAM Role Setup for CloudWatch**

Attach IAM role to EC2:

Policy:

AmazonCloudWatchReadOnlyAccess

**Steps:**

Go to IAM → Roles
Create Role → EC2
Attach CloudWatch ReadOnlyAccess & CloudWatchLogsReadOnlyAccess
Attach role to EC2 instance
📡 Configure CloudWatch Data Source

**In Grafana UI:**

Go to Settings → Data Sources
Add data source → CloudWatch
Authentication:
Use IAM Role (recommended)
Region: ap-south-1
Save & Test
📊 Dashboards Created
1. EC2 Instance Monitoring Dashboard

Metrics:

CPU Utilization
Network In/Out
Disk Read/Write
2. System Health Dashboard

Metrics:

Memory usage (via CloudWatch agent)
Disk usage
System status checks
3. Application Performance Dashboard

Metrics:

Latency
Request count
Error rate (4xx/5xx)
4. AWS Infrastructure Overview Dashboard

Metrics:

EC2 fleet health
Load balancer metrics
CloudWatch alarms
📈 Sample CloudWatch Metrics Queries
CPU Utilization
Namespace: AWS/EC2
Metric: CPUUtilization
Statistic: Average
Network In
Namespace: AWS/EC2
Metric: NetworkIn
Statistic: Sum

🔥 **Example SRE Dashboard Panels**
RED metrics (Rate, Errors, Duration)
CPU vs Memory correlation
Top 5 highest CPU instances
API error spikes
Latency p95/p99 trends

🔒 **Security Best Practices**
Restrict port 3000 to your IP
Disable public SSH access (use bastion host)
Use IAM roles instead of access keys
Enable HTTPS using reverse proxy
Store dashboards in Git (Infrastructure as Code)

⚙️ **Troubleshooting**
Grafana not accessible
sudo ss -tulpn | grep 3000
Service not running
sudo systemctl restart grafana-server
sudo journalctl -u grafana-server -f
CloudWatch data missing
Verify IAM role attached
Check region mismatch
Ensure metrics exist in CloudWatch

🧠 **SRE Learning Outcomes**

This setup demonstrates:

**Observability architecture**
Cloud-native monitoring
AWS-native metrics integration
Dashboard design for production systems
Incident detection readiness

🚀 **Future Enhancements**
Add Prometheus as secondary datasource
Integrate Loki for logs
Add alerting to Slack / PagerDuty
Deploy using Terraform
Enable HTTPS using Nginx + Let’s Encrypt

📌 **Repository Structure**
grafana-ec2-cloudwatch/
│
├── README.md
├── dashboards/
│   ├── ec2-monitoring.json
│   ├── system-health.json
│   └── aws-overview.json
├── provisioning/
│   ├── datasources.yaml
│   └── dashboards.yaml
└── scripts/
    └── install-grafana.sh
🧾 License

This project is for learning and demonstration purposes.
