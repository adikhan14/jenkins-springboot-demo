# 01 — EC2 Instance Setup

## Launch EC2 Instance

1. Go to **AWS Console → EC2 → Launch Instance**
2. Configure:

| Setting | Value |
|---------|-------|
| **OS / AMI** | Ubuntu |
| **Instance Type** | Large (t2.large / t3.large) |
| **Key Pair** | Create new → download `.pem` file |
| **Security Group** | Create new (configure below) |
| **Storage** | **Minimum 20GB** root volume |

> ⚠️ 8GB is NOT enough — Jenkins + Docker + SonarQube will fill it up fast!

---

## Security Group Inbound Rules

| Port | Protocol | Purpose |
|------|----------|---------|
| 22 | TCP | SSH access |
| 8080 | TCP | Jenkins UI |
| 9000 | TCP | SonarQube UI |

---

## Expand EBS Volume (If Needed)

If you launched with 8GB and ran out of space:

1. **AWS Console → EC2 → Elastic Block Store → Volumes**
2. Select root volume → **Actions → Modify Volume**
3. Change to **20 GB** → **Modify**

Then on EC2:
```bash
sudo growpart /dev/xvda 1
sudo resize2fs /dev/root
df -h
```

---

## SSH into EC2

```bash
# Set permissions on PEM file
chmod 400 aws-pem-file.pem

# Connect
ssh -i "aws-pem-file.pem" ubuntu@65.1.134.229
```

> ⚠️ If you get `Operation timed out` → Security Group port 22 is not open. Add SSH inbound rule.

---

## Instance Details

| Detail | Value |
|--------|-------|
| **Public IP** | 65.1.134.229 |
| **Private IP** | 172.31.10.167 |
| **OS** | Ubuntu |
| **Key Pair** | aws-pem-file.pem |
