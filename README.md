# Cloud Database Deployment & CRUD Verification (AWS RDS MySQL & DynamoDB)

**Author:** Kinjal Kapuraram Goyal  
**Institution:** Fr. Conceicao Rodrigues College of Engineering, Mumbai  
**Student / Lab Identifier:** `10732`  
**AWS Region:** Asia Pacific (Mumbai) — `ap-south-1`  

---

## 1. Architecture & Resource Inventory

This project demonstrates relational and NoSQL cloud database deployment on AWS, connecting a Python Flask application hosted on Amazon EC2 to a private Amazon RDS MySQL 8.4 instance over TLS/SSL and an Amazon DynamoDB table authenticated via a least-privilege IAM instance profile.

| Component | AWS Service | Resource Identifier / Configuration |
| :--- | :--- | :--- |
| **Application Host** | Amazon EC2 (`t3.micro`) | `docker-lab-10732` (`i-0f11604fc4a3c5204`) |
| **Security Group** | Amazon VPC Security Group | `docker-lab-sg` (`sg-06475091a3978240c`) — Port `5000` (HTTP), Port `22` (SSH) |
| **Relational Database** | Amazon RDS (MySQL 8.4.11) | `lab5-mysql-10732` (`db.t3.micro`, `Publicly accessible: No`, TLS enforced) |
| **Database Schema** | MySQL (`college_registration`) | Normalized tables: `departments` & `students` with foreign-key constraint |
| **NoSQL Database** | Amazon DynamoDB | `Students10732` (Partition key: `student_id` [String], On-Demand capacity) |
| **Access Control** | AWS IAM Role & Inline Policy | `ec2-dynamodb-role` with `students-table-access` (scoped to `Students10732`) |

---

## 2. Part A: Relational Database (Amazon RDS MySQL)

### Infrastructure & Security Configuration
* **Network Isolation:** Deployed `lab5-mysql-10732` with **Publicly accessible: No**, restricting inbound port `3306` exclusively to the EC2 security group (`ec2-rds-1`).
* **Encrypted Transport:** Verified connections using AWS RDS global CA bundle (`global-bundle.pem`) with `--ssl-verify-server-cert`.
* **Principle of Least Privilege:** Created a dedicated database user (`appuser`) restricted to `SELECT, INSERT, UPDATE, DELETE` on `college_registration.*` with `REQUIRE SSL`.

### Schema Design (`college_registration`)
```sql
CREATE DATABASE IF NOT EXISTS college_registration;
USE college_registration;

CREATE TABLE departments (
  dept_id INT AUTO_INCREMENT PRIMARY KEY,
  dept_name VARCHAR(60) NOT NULL UNIQUE
);

CREATE TABLE students (
  student_id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(120) NOT NULL UNIQUE,
  phone VARCHAR(15),
  major_dept_id INT NOT NULL,
  gpa DECIMAL(4,2),
  is_active BOOLEAN DEFAULT TRUE,
  registered_on TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (major_dept_id) REFERENCES departments(dept_id),
  INDEX idx_major (major_dept_id)
);
