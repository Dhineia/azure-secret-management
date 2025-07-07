## Azure Key Vault Integration for GitHub Actions
Securely retrieve secrets from Azure Key Vault using GitHub Actions and federated identity with zero-trust access control.
---
Architecture Overview
![image](https://github.com/user-attachments/assets/110b88d6-04c6-429a-adb9-2abdb10b594f)

---
## Step-by-Step Setup
1. Create Azure Key Vault
- Name: myKeyVault-01
- Location: eastus
- Resource group: secrets-management

Create a secret:
az keyvault secret set \
  --name app-secret \
  --vault-name myKeyVault-01 \
  --value "superSecretValue123"
  
Add Role Assignment to Key Vault
1. Go to Your Key Vault
- Azure Portal → Navigate to Key Vaults
- Select myKeyVault-01
2. Open Access Control (IAM)
- In the left panel, click Access Control (IAM)
- Click the “Role assignments” tab (top navigation)
3. Add a Role Assignment
Click + Add → Add role assignment
Fill out the following:
![image](https://github.com/user-attachments/assets/d7042954-d167-46ea-815b-d0720aa7cd01)

2. Register App for GitHub Federated Identity
- Azure Portal → Azure AD → App registrations → New registration
- Name: github-federated-access
- Leave redirect URI empty
After registration:
- Go to Federated credentials tab → Add credential:
- Issuer: https://token.actions.githubusercontent.com
- Subject: repo:<org>/<repo>:ref:refs/heads/main
- Audience: api://AzureADTokenExchange

3. Assign RBAC Roles to GitHub Identity
Go to myKeyVault-01 → Access Control (IAM) → Add role assignment:
![image](https://github.com/user-attachments/assets/85fbc8c2-432f-472a-862a-e118538863fa)

5. GitHub Workflow Example
Store these secrets in your repo (AZURE_CLIENT_ID, AZURE_TENANT_ID, AZURE_SUBSCRIPTION_ID).
Sample workflow:
jobs:
  fetch-azure-secret:
    runs-on: ubuntu-latest

    permissions:
      id-token: write
      contents: read

    steps:
    - name: Azure Login
      uses: azure/login@v1
      with:
        client-id: ${{ secrets.AZURE_CLIENT_ID }}
        tenant-id: ${{ secrets.AZURE_TENANT_ID }}
        subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

    - name: Retrieve secret
      run: |
        SECRET=$(az keyvault secret show \
          --name app-secret \
          --vault-name myKeyVault-01 \
          --query value -o tsv)
        echo "SECRET=$SECRET" >> $GITHUB_ENV
      







