
# 🛡️ Complete Ubuntu Setup Guide for WAF: PostgreSQL & Redis

> A comprehensive, step-by-step guide to install and configure PostgreSQL and Redis on Ubuntu for your Web Application Firewall (WAF)

[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04%20|%2020.04-orange?style=flat-square&logo=ubuntu)](https://ubuntu.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-14%2B-blue?style=flat-square&logo=postgresql)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Redis-7%2B-red?style=flat-square&logo=redis)](https://redis.io/)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

---

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [PostgreSQL Installation](#-postgresql-installation)
- [Redis Installation](#-redis-installation)
- [Firewall Configuration](#-firewall-configuration)
- [Final Verification](#-final-verification)
- [WAF Configuration](#-update-your-waf-configuration)
- [Quick Reference](#-quick-reference-commands)
- [Troubleshooting](#-troubleshooting)

---

## 📋 Prerequisites

Update your system packages:

```bash
# Update package lists
sudo apt update

# Upgrade installed packages
sudo apt upgrade -y
🐘 PostgreSQL Installation
Step 1: Install PostgreSQL
bash
# Install PostgreSQL server and contrib packages
sudo apt install -y postgresql postgresql-contrib
Step 2: Verify Installation
bash
# Check service status
sudo systemctl status postgresql

# Check version
psql --version
Expected output:

text
psql (PostgreSQL) 14.12 (Ubuntu 14.12-0ubuntu0.22.04.1)
Step 3: Access PostgreSQL
bash
# Switch to postgres user
sudo -i -u postgres

# Start PostgreSQL prompt
psql
You should see:

text
postgres=#
Step 4: Set Postgres User Password
At the postgres=# prompt:

sql
-- Replace 'your_secure_password' with your actual password
ALTER USER postgres WITH ENCRYPTED PASSWORD 'your_secure_password';

-- Exit PostgreSQL
\q
Return to your normal user:

bash
exit
Step 5: Create WAF Database
bash
# Connect to PostgreSQL
sudo -u postgres psql
At the postgres=# prompt:

sql
-- Create database
CREATE DATABASE waf_db;

-- List databases to verify
\l

-- Exit
\q
Expected output:

text
List of databases
   Name    |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   
-----------+----------+----------+---------+---------+-----------------------
 postgres  | postgres | UTF8     | en_US   | en_US   | 
 template0 | postgres | UTF8     | en_US   | en_US   | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US   | en_US   | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
 waf_db    | postgres | UTF8     | en_US   | en_US   | 
(4 rows)
Step 6: Configure Password Authentication
Find your PostgreSQL version:

bash
psql --version
Edit the authentication configuration (replace 14 with your version):

bash
sudo nano /etc/postgresql/14/main/pg_hba.conf
Find this line:

text
local   all             all                                     peer
Change it to:

text
local   all             all                                     md5
For PostgreSQL 15+, use:

text
local   all             all                                     scram-sha-256
Save the file: Ctrl+O, Enter, Ctrl+X

Step 7: Restart PostgreSQL
bash
# Apply changes
sudo systemctl restart postgresql

# Verify restart
sudo systemctl status postgresql
Step 8: Test Connection
bash
# Test with password authentication
psql -U postgres -d waf_db -h localhost
Enter your password when prompted. You should see:

text
Password for user postgres: 
psql (14.12 (Ubuntu 14.12-0ubuntu0.22.04.1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384)
Type "help" for help.

waf_db=#
Exit:

sql
\q
🔴 Redis Installation
Step 1: Install Redis
bash
# Update package index
sudo apt update

# Install Redis server
sudo apt install -y redis-server
Step 2: Verify Installation
bash
# Check service status
sudo systemctl status redis-server

# Check version
redis-server --version
Expected output:

text
Redis server v=7.0.15 sha=00000000:0 malloc=jemalloc-5.3.0 bits=64 build=6a39f1b58b8c27a6
Step 3: Test Basic Functionality
bash
# Start Redis CLI
redis-cli
At the 127.0.0.1:6379> prompt:

redis
# Test connection
ping
Expected output:

text
PONG
redis
# Test key-value operations
set test "Hello WAF"
Expected output:

text
OK
redis
get test
Expected output:

text
"Hello WAF"
Exit:

redis
exit
Step 4: Configure Redis
Backup the original configuration:

bash
sudo cp /etc/redis/redis.conf /etc/redis/redis.conf.backup
Edit Redis configuration:

bash
sudo nano /etc/redis/redis.conf
Make these changes:

a) Set memory limit:

conf
# Find and uncomment/modify this line
maxmemory 256mb
b) Set memory policy:

conf
# Find and uncomment/modify this line
maxmemory-policy allkeys-lru
c) Optional: Set a password:

conf
# Find and uncomment/modify this line
requirepass your_redis_password
d) Bind to localhost (keep for security):

conf
# Ensure this line is uncommented
bind 127.0.0.1
Save the file: Ctrl+O, Enter, Ctrl+X

Step 5: Restart Redis
bash
# Apply changes
sudo systemctl restart redis-server

# Verify restart
sudo systemctl status redis-server
Step 6: Test with Password (If Set)
bash
# Connect to Redis
redis-cli
At the 127.0.0.1:6379> prompt:

redis
# Authenticate
AUTH your_redis_password
Expected output:

text
OK
redis
# Test connection
ping
Expected output:

text
PONG
Exit:

redis
exit
Step 7: Enable Auto-Start
bash
# Enable on boot
sudo systemctl enable redis-server

# Verify
sudo systemctl is-enabled redis-server
Expected output:

text
enabled
🔧 Firewall Configuration
If UFW is active:

bash
# Check UFW status
sudo ufw status
If you need remote access (NOT recommended for local development):

bash
# Allow PostgreSQL port
sudo ufw allow 5432/tcp comment 'PostgreSQL'

# Allow Redis port
sudo ufw allow 6379/tcp comment 'Redis'

# Reload firewall
sudo ufw reload
⚠️ SECURITY WARNING: Only open these ports if absolutely necessary. For local development, keep them closed.

✅ Final Verification
Check PostgreSQL
bash
sudo systemctl status postgresql --no-pager
Check Redis
bash
sudo systemctl status redis-server --no-pager
Test PostgreSQL Connection
bash
psql -U postgres -d waf_db -h localhost -c "SELECT 1"
Enter password. Expected output:

text
 ?column? 
----------
        1
(1 row)
Test Redis Connection
With password:

bash
redis-cli -a your_redis_password ping
Expected output: PONG

Without password:

bash
redis-cli ping
Expected output: PONG

📝 Update Your WAF Configuration
Edit your Capstone35.py file:

python
# ============================================================================
# CONFIGURATION SECTION - Update these lines
# ============================================================================

# PostgreSQL connection string
# Replace YOUR_POSTGRES_PASSWORD with the password you set
POSTGRES_DSN = os.getenv(
    "POSTGRES_DSN", 
    "postgresql://postgres:YOUR_POSTGRES_PASSWORD@localhost/waf_db"
)

# Redis connection string - WITH password
REDIS_URL = os.getenv(
    "REDIS_URL", 
    "redis://:YOUR_REDIS_PASSWORD@localhost:6379/0"
)

# Redis connection string - WITHOUT password (uncomment if no password set)
# REDIS_URL = os.getenv("REDIS_URL", "redis://localhost:6379/0")












