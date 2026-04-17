Act as a senior cloud platform architect and full-stack engineer. Design and generate a production-ready MVP for an internal AWS account vending portal.

Requirements:
- Python backend (prefer FastAPI)
- Simple enterprise UI
- Microsoft Entra ID authentication via OIDC
- Terraform-based AWS account vending
- Terraform state locking/tracking in DynamoDB
- Optional S3 remote backend for Terraform state
- Multi-user internal portal
- On-demand AWS account provisioning

Form fields:
- TeamName
- Env
- System ID
- POC

Naming rules:
- Account name: gw-<team_name>-<env>-001
- Account email: gw_<team_name>_<env>_001_aws_acctmgmt@htl.com

Features:
- Submit account vending requests
- Provision AWS accounts via Terraform
- Track request status
- View existing accounts with full metadata
- Show AWS OU structure
- Export inventory to CSV

Please provide:
1. Architecture diagram explanation
2. Component breakdown
3. Backend code structure
4. Frontend page layout
5. API design
6. Entra ID auth integration
7. Terraform execution workflow
8. AWS Organizations integration model
9. CSV export implementation
10. Security, logging, RBAC, and audit recommendations
11. Deployment recommendations
12. Sample code for the MVP
