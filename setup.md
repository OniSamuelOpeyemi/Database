Connect from EC2 to RDS

SSH into EC2 (if public SSH allowed):
```bash
ssh -i mykey.pem ubuntu@{ec2-public-ip}

```

Install Postgres client (Ubuntu example):

```bash
sudo apt update
sudo apt install -y postgresql-client
```

Connect to RDS:

```bash
# Option 1: set env var for password and run psql
bash
export PGPASSWORD='YourRdsPassword'
psql -h your-rds-endpoint.rds.amazonaws.com -U masteruser -d postgres -p 5432

# Option 2: using SSL (recommended)
psql "host=your-rds-endpoint.rds.amazonaws.com port=5432 dbname=postgres user=masteruser sslmode=require"

```

For MySQL:

```bash
sudo apt install -y default-mysql-client
mysql -h your-rds-endpoint.rds.amazonaws.com -P 3306 -u admin -p

```

Tip: use ~/.pgpass for password storage:

```bash
echo "your-rds-endpoint.rds.amazonaws.com:5432:postgres:masteruser:YourRdsPassword" >> ~/.pgpass
chmod 600 ~/.pgpass
psql -h your-rds-endpoint.rds.amazonaws.com -U masteruser -d postgres

```
E. If RDS is in private subnets and you’re developing from your laptop

Options:

Bastion host (jump box) in public subnet; SSH tunnel:

```bash
ssh -i key.pem -L 5432:your-rds-endpoint.rds.amazonaws.com:5432 ec2-user@bastion-public-ip -N
# then on local machine:
psql -h localhost -p 5432 -U masteruser -d postgres

```
AWS Systems Manager (Session Manager): connect to EC2 without SSH keys; prefer this for security (no open inbound ports).

VPN or Direct Connect: for production on-prem connectivity, set up Site-to-Site VPN or Direct Connect to your VPC (secure link).

F. Example: create a test table & query (Postgres)

```bash
CREATE TABLE customers (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  email TEXT UNIQUE,
  created_at TIMESTAMP DEFAULT now()
);

INSERT INTO customers (name, email) VALUES ('Jane Doe','jane@example.com');

SELECT * FROM customers;

```
G. Create a read replica (basic idea)

Console: RDS → your DB → Actions → Create read replica. Choose instance type, subnet group.

Use read replica endpoints for read workloads (reporting, analytics). Watch for replication lag.