# Create AWS VPC 

## Block Diagram

<img src="img/awsvpc.png" height="400" width="500">


Below are the 10 steps to Create and Verify your VPC

* Step 01. Create a VPC

* Step 02. Create 2 Public Subnet & Create 2 Private Subnet

* Step 03. Create IGW (Internet Gateway) & Attach to the VPC

* Step 04. Create Public and Private Route Table

* Step 05. Add IGW in Public Route table (0.0.0.0/0)

* Step 06. Add Public Subnet (1a & 1b) in Route table

* Step 07. Create a NAT Gateway in Public Subnet

* Step 08. Add NAT GW into the Private Route Table

* Step 09. Add Private Subnet in Private Route Table

* Step 10. Launch EC2 in this VPC & Validate your Connection
