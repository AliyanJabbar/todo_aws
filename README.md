# AWS Todo App Setup and Running Guide

This guide will help you set up and run the **Todo App** on an AWS EC2 instance connected to an RDS PostgreSQL database.

---

## Prerequisites

* AWS account
* Basic knowledge of EC2 and RDS

---

## Steps to Set Up

### 1. Create an IAM Role for EC2

* Go to **IAM** in the AWS Console.
* Create a new role with **EC2** as the trusted entity.
* Attach the following policies:

  * AmazonS3FullAccess (for S3 log uploads)
  * AmazonRDSFullAccess (or appropriate RDS access policy)
* Give it a name (e.g., `EC2-TodoApp-Role`) and create the role.
* This IAM role will be automatically used by the app to push logs to S3 without manually setting AWS credentials.

### 2. Create an EC2 Instance

* Go to the **EC2** service in the AWS Console.
* Launch a new instance with your preferred OS (e.g., Ubuntu).
* Assign the IAM Role created in Step 1 to the instance.
* In **Security Groups**, allow inbound access to:

  * SSH (port 22)
  * HTTP (port 80) if needed
  * Custom TCP (port 8000) for FastAPI
* Review and launch the instance.

### 3. Create an RDS Instance

* Go to the **RDS** service in AWS.
* Launch a new **PostgreSQL** database.
* Configure username, password, and database name.
* Allow your EC2 instance to access the RDS database:

  * In RDS **Connectivity**, choose **VPC security groups** and add the EC2 security group.

### 4. Connect EC2 with RDS

* In the **AWS Console**, ensure your EC2 instance can connect to the RDS database.
* Use the RDS endpoint, username, and password for connection.

### 5. Connect to Your EC2 Instance

* Go to your EC2 instance and click **Connect**.
* Open the terminal via **EC2 Instance Connect**.

### 6. Install PostgreSQL Client on EC2

```bash
sudo apt update
sudo apt install postgresql-client -y
```

### 7. Install `uv` on EC2

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### 8. Clone This Repository

```bash
git clone <your-repo-url>
```

### 9. Navigate to Your Project Directory

```bash
cd todo_aws
```

### 10. Create a Virtual Environment

```bash
uv venv
```

### 11. Install Dependencies

```bash
uv add "fastapi[standard]"
```

> This command installs all dependencies listed in `pyproject.toml`.

### 12. Start FastAPI Server

```bash
fastapi dev main.py --host 0.0.0.0
```

### 13. Access Your App

* Open a new browser tab and navigate to:

```
http://<your-ubuntu-ec2-public-ip>:8000
```

### 14. Assign IAM Role for S3 Access

* Ensure the EC2 instance has the IAM role created in Step 1 assigned.
* This role allows the app to automatically push logs to S3 without configuring AWS credentials manually.

### 15. Logging and S3 Upload

* The app automatically creates logs in `app.log` in the project root.
* Logs are **pushed to S3 every 10 seconds** (bucket: `todoappkis3`) under the `logs/` folder with a timestamp:

```
logs/app_log_2025-11-09_16-50-10.log
logs/app_log_2025-11-09_16-50-20.log
```

* You can manually upload logs using the endpoint:

```bash
curl -X POST http://<your-ec2-public-ip>:8000/upload-logs
```

* Or using the AWS CLI:

```bash
aws s3 cp app.log s3://todoappkis3/logs/app_log_$(date +%Y-%m-%d_%H-%M-%S).log
```

> The app uses the EC2 **IAM Role** for S3 access — no manual credentials are needed.

### 16. Set CloudWatch Alarm for High Load

* Go to **ec2 > instance > Alarm Status on table's header**.
* Create a new alarm monitoring **EC2 CPUUtilization**.
* Set the threshold to **>= 0.99%** over **1 consecutive 5-minute period**.
* Attach an **SNS Topic** for email notifications when the alarm triggers.

### 17. Add Load Balancer (Optional) (not for free tier)

* Go to **EC2 > Load Balancers**.
* Create an **Application Load Balancer (ALB)**.
* Add your EC2 instance to the target group.
* Update security groups and listeners to forward HTTP/HTTPS traffic to your instance.

---

## Notes

* Make sure the EC2 security group allows inbound traffic on port 8000.
* For production, consider using **NGINX** or **ALB** in front of FastAPI.
* Keep your RDS credentials secure and do not commit them to the repository.
* The log uploader will continuously push logs to S3 every 10 seconds while the app is running.
* CloudWatch alarms can alert you via email when CPU usage is high.
