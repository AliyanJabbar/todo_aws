# AWS Todo App Setup and Running Guide

This guide will help you set up and run the **Todo App** on an AWS EC2 instance connected to an RDS PostgreSQL database.

---

## Prerequisites

* AWS account
* Basic knowledge of EC2 and RDS
* Git installed on your local machine

---

## Steps to Set Up

### 1. Create an EC2 Instance

* Go to the **EC2** service in the AWS Console.
* Launch a new instance with your preferred OS (e.g., Ubuntu).
* In **Security Groups**, allow inbound access to:

  * SSH (port 22)
  * HTTP (port 80) if needed
  * Custom TCP (port 8000) for FastAPI
* Review and launch the instance.

### 2. Create an RDS Instance

* Go to the **RDS** service in AWS.
* Launch a new **PostgreSQL** database.
* Configure username, password, and database name.
* Allow your EC2 instance to access the RDS database:

  * In RDS **Connectivity**, choose **VPC security groups** and add the EC2 security group.

### 3. Connect EC2 with RDS

* In the **AWS Console**, ensure your EC2 instance can connect to the RDS database.
* Use the RDS endpoint, username, and password for connection.

### 4. Connect to Your EC2 Instance

* Go to your EC2 instance and click **Connect**.
* Open the terminal via **EC2 Instance Connect**.

### 5. Install PostgreSQL Client on EC2

```bash
sudo apt update
sudo apt install postgresql-client -y
```

### 6. Install `uv` on EC2

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### 7. Clone This Repository

```bash
git clone <your-repo-url>
```

### 8. Navigate to Your Project Directory

```bash
cd todo_aws
```

### 9. Create a Virtual Environment

```bash
uv venv
```

### 10. Install Dependencies

```bash
uv add "fastapi[standard]"
```

> This command installs all dependencies listed in `pyproject.toml`.

### 11. Start FastAPI Server

```bash
fastapi dev main.py --host 0.0.0.0
```

### 12. Access Your App

* Open a new browser tab and navigate to:

```
http://<your-ubuntu-ec2-public-ip>:8000
```

### 13. All Set!

* Your Todo App should now be running and connected to the PostgreSQL RDS database.

---

## Notes

* Make sure the EC2 security group allows inbound traffic on port 8000.
* For production, consider using **NGINX** or **ALB** in front of FastAPI.
* Keep your RDS credentials secure and do not commit them to the repository.
