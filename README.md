# vpc-task-2
# AWS VPC Setup: Public + Private Subnet

This project sets up an AWS VPC with a public subnet and a private subnet across multiple Availability Zones (AZs), alongside an Internet Gateway for outbound internet access.

---

## 🎯 Objectives

- Create a new VPC in any AWS region  
- Create **2 subnets** in different AZs:
  - **Public subnet** for internet-facing resources  
  - **Private subnet** for internal-only resources  
- Attach an **Internet Gateway (IGW)** to the VPC  
- Configure **Route Tables** to enable internet access:
  - Public subnet → IGW  
  - Private subnet → no direct internet access  

---

## ✅ What You Did

1. **Created a VPC** with a specified CIDR block (e.g., `10.0.0.0/16`)  
2. **Created 2 subnets** in separate AZs within the VPC:  
   - `10.0.1.0/24` (public)  
   - `10.0.2.0/24` (private)  
3. **Provisioned** an Internet Gateway and attached it to the VPC :contentReference[oaicite:1]{index=1}  
4. **Configured route tables**:
   - **Public Route Table**: Configured route `0.0.0.0/0` → IGW, associated with public subnet :contentReference[oaicite:2]{index=2}
   - **Private Route Table**: Default or explicit, no IGW route

---

## 🛠️ CLI Setup Steps

```bash
# 1. Create VPC
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output text)

# 2. Create Public Subnet (AZ A)
PUBLIC_SUBNET=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --query Subnet.SubnetId \
  --output text)

# 3. Create Private Subnet (AZ B)
PRIVATE_SUBNET=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1b \
  --query Subnet.SubnetId \
  --output text)

# 4. Create and attach Internet Gateway
IGW_ID=$(aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text)
aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $IGW_ID

# 5. Create Public Route Table and route
RTB_PUBLIC=$(aws ec2 create-route-table --vpc-id $VPC_ID --query RouteTable.RouteTableId --output text)
aws ec2 create-route --route-table-id $RTB_PUBLIC --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID

# 6. Associate Public Subnet
aws ec2 associate-route-table --route-table-id $RTB_PUBLIC --subnet-id $PUBLIC_SUBNET
