# ROBOSHOP SCRIPT:
* $@-->pass all arguments from outside when run script 
$1 = mongodb
$2 = redis
$3 = mysql
ex: sh mongodb catalgue,redis,cart ..etc 

for instance in $@ -->which meanis 

we run mongodb redis cart we run 3 times so we can use loop repeated tasks we use loop 



# AWS EC2 and Route 53 Automation Script

## 1. Define AMI ID

```bash
AMI_ID="ami-0220d79f3f480ecf5"
```

This variable stores the AMI ID used to create EC2 instances.

---

## 2. Define Security Group ID

```bash
SG_ID="sg-0f6fd0ebbf2749559"
```

This variable stores the Security Group ID used for the EC2 instances.

---

## 3. Define Instance Names

```bash
INSTANCES=("mongodb" "redis" "mysql" "rabbitmq" "catalogue" "user" "cart" "shipping" "payment" "dispatch" "frontend")
```

This contains the names of all required EC2 instances.

---

## 4. Define Route 53 Hosted Zone ID

```bash
ZONE_ID="Z05169793PXSJ01X7N5ZV"
```

This stores the Route 53 Hosted Zone ID.

---

## 5. Define Domain Name

```bash
DOMAIN_NAME="kodari.online"
```

This stores the domain name used for DNS records.

---

## 6. Get Instance Names from Arguments

```bash
for instance in $@
```

`$@` means all arguments passed to the script.

Example:

```bash
./script.sh mongodb redis mysql
```

Here:

```text
mongodb
redis
mysql
```

are passed as arguments.

---

## 7. Create EC2 Instance

```bash
INSTANCE_ID=$(aws ec2 run-instances \
--image-id ami-0220d79f3f480ecf5 \
--instance-type t3.micro \
--security-group-ids sg-03f077f464f434b47 \
--tag-specifications "ResourceType=instance,Tags=[{Key=Name, Value=$instance}]" \
--query "Instances[0].InstanceId" \
--output text)
```

This command creates an EC2 instance.

It uses:

* AMI ID
* `t3.micro` instance type
* Security Group
* Instance Name tag

The created Instance ID is stored in:

```bash
INSTANCE_ID
```

---

## 8. Check if Instance is Frontend

```bash
if [ $instance != "frontend" ]
```

This checks whether the instance name is not `frontend`.

---

## 9. Get Private IP

```bash
IP=$(aws ec2 describe-instances \
--instance-ids $INSTANCE_ID \
--query "Reservations[0].Instances[0].PrivateIpAddress" \
--output text)
```

For backend services, this gets the **Private IP address** of the EC2 instance.

Example:

```text
mongodb → 172.31.20.10
redis   → 172.31.20.11
```

---

## 10. Create Backend DNS Name

```bash
RECORD_NAME="$instance.$DOMAIN_NAME"
```

This creates a subdomain.

Example:

```text
mongodb.kodari.online
redis.kodari.online
mysql.kodari.online
```

---

## 11. Frontend Public IP

```bash
else
```

If the instance is `frontend`, the script executes this section.

```bash
IP=$(aws ec2 describe-instances \
--instance-ids $INSTANCE_ID \
--query "Reservations[0].Instances[0].PublicIpAddress" \
--output text)
```

It gets the **Public IP address** of the frontend instance.

---

## 12. Frontend DNS Name

```bash
RECORD_NAME="$DOMAIN_NAME"
```

The frontend uses the main domain.

Example:

```text
kodari.online
```

---

## 13. Print IP Address

```bash
echo "$instance IP address: $IP"
```

This prints the instance name and IP address.

Example:

```text
mongodb IP address: 172.31.20.10
frontend IP address: 54.123.45.67
```

---

## 14. Create Route 53 Record

```bash
aws route53 change-resource-record-sets \
--hosted-zone-id $ZONE_ID \
--change-batch
```

This command creates or updates a DNS record in Route 53.

---

## 15. Use UPSERT

```json
"Action": "UPSERT"
```

UPSERT means:

* Create the record if it does not exist.
* Update the record if it already exists.

---

## 16. Create A Record

```json
"Type": "A"
```

An A record maps a domain name to an IPv4 address.

Example:

```text
mongodb.kodari.online → 172.31.20.10
```

---

## 17. Set IP Address

```json
"Value": "'$IP'"
```

This adds the EC2 instance IP address to the Route 53 record.

---

## 18. Complete Flow

```text
Script Arguments
      ↓
Create EC2 Instance
      ↓
Get Instance ID
      ↓
Get IP Address
      ↓
Check Frontend or Backend
      ↓
Create DNS Record Name
      ↓
Route 53 UPSERT
      ↓
DNS Record Created/Updated
```

## Run the Script

```bash
chmod +x script.sh
```

Run with selected services:

```bash
./script.sh mongodb redis mysql
```

Run all services:

```bash
./script.sh mongodb redis mysql rabbitmq catalogue user cart shipping payment dispatch frontend
```

## Example DNS Records

```text
mongodb.kodari.online → Private IP
redis.kodari.online   → Private IP
mysql.kodari.online   → Private IP
rabbitmq.kodari.online → Private IP
catalogue.kodari.online → Private IP
user.kodari.online → Private IP
cart.kodari.online → Private IP
shipping.kodari.online → Private IP
payment.kodari.online → Private IP
dispatch.kodari.online → Private IP

kodari.online → Frontend Public IP
```

R → Red
G → Green
Y → Yellow
N → Normal