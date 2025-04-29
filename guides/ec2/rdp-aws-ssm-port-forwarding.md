# Remote Desktop Connection (RDP) via AWS SSM Port Forwarding

This guide provides a step-by-step process to connect to a Windows EC2 instance using **Remote Desktop Protocol (RDP)** via **AWS Systems Manager Session Manager** with **Port Forwarding**. This method enhances security by eliminating the need to open port `3389` to the internet.

---

## 📌 Prerequisites
Before proceeding, ensure you have the following:
- ✅ An **AWS EC2 Windows instance** with **SSM Agent installed and running**.
- ✅ Proper **IAM permissions** for SSM Session Manager.
- ✅ AWS CLI installed and configured with the necessary permissions.
- ✅ RDP client installed on your local machine.

---

## 🚀 Step 1: Verify SSM Agent is Running

### On the EC2 Instance (Windows)
Run the following command in **PowerShell** to check if the **SSM Agent** is running:
```powershell
Get-Service AmazonSSMAgent
```
If the agent is not running, start it:
```powershell
Start-Service AmazonSSMAgent
```
If the agent is not installed, download and install it from AWS documentation.

---

## 🔑 Step 2: Create a New User on the Windows Machine
If you need to create a new **Windows user** for RDP, run the following PowerShell script **on the EC2 instance**:

```powershell
$UserName = "NewUser"
$Password = ConvertTo-SecureString "StrongPassword123!" -AsPlainText -Force
New-LocalUser -Name $UserName -Password $Password -FullName "New User" -Description "User for RDP access"
Add-LocalGroupMember -Group "Remote Desktop Users" -Member $UserName
Write-Host "User $UserName created and added to Remote Desktop Users group."
```
🔹 Replace `NewUser` with your desired username.  
🔹 Replace `StrongPassword123!` with a secure password.

---

## 🌍 Step 3: Configure IAM Role for EC2

If your EC2 instance is not already associated with an **IAM role**, you need to create one with the following permissions:

### 1️⃣ **Create an IAM Role**
- Go to **AWS IAM Console** > **Roles** > **Create Role**.
- Select **AWS Service** → **EC2**.
- Attach the following policy:
  ```json
  {
    "Effect": "Allow",
    "Action": [
      "ssm:StartSession",
      "ssm:DescribeSessions",
      "ssm:TerminateSession",
      "ec2:DescribeInstances",
      "ssm:GetConnectionStatus"
    ],
    "Resource": "*"
  }
  ```
- Name the role (e.g., `SSMAccessRole`) and attach it to the EC2 instance.

### 2️⃣ **Attach the Role to the EC2 Instance**
- Open **EC2 Console**.
- Select your instance.
- Click **Actions** > **Modify IAM Role**.
- Select the new role and attach it.

---

## 🌍 Step 4: Install AWS CLI and Session Manager Plugin

To enable session management and port forwarding, you need both the **AWS CLI** and the **Session Manager Plugin** installed on your local machine.

### ➡️ Install AWS CLI:

1. Download the AWS CLI installer for Windows:  
   [Download AWS CLI v2 for Windows (.msi)](https://awscli.amazonaws.com/AWSCLIV2.msi)

2. Run the installer and complete the setup.

3. Verify the installation:
   ```sh
   aws --version
You should see the AWS CLI version output.

### ➡️ Install Session Manager Plugin:

1. Download the Session Manager Plugin installer for Windows:  
   [Download Session Manager Plugin](https://docs.aws.amazon.com/systems-manager/latest/userguide/install-plugin-windows.html)

2. Run the installer and complete the setup.

3. Verify the plugin installation:
    ```sh
    session-manager-plugin --version
You should see the plugin version output.

---

## 🔑 Step 5: Configure Your Local Machine

After installing the AWS CLI and the Session Manager Plugin, configure the AWS CLI with your credentials:
    
    aws configure
Ensure that the credentials used have permissions to start SSM sessions.

---

## 🎯 Step 6: Start Port Forwarding Session
Now, use the AWS CLI to establish an **SSM session with Port Forwarding**:

```sh
aws ssm start-session --target <INSTANCE_ID> --document-name AWS-StartPortForwardingSession --parameters "portNumber=3389,localPortNumber=13389"
```
🔹 Replace `<INSTANCE_ID>` with your EC2 instance ID.  
🔹 `portNumber=3389` is the RDP port on the EC2 instance.  
🔹 `localPortNumber=13389` is the port on your machine (you can change this if needed).

If successful, you should see:
```
Starting session with SessionId: ...
Port 13389 on your local machine is forwarding to port 3389 on the remote instance.
```

---

## 🖥 Step 7: Connect to RDP via Localhost

1️⃣ Open **Remote Desktop Connection** (`mstsc.exe` on Windows).  
2️⃣ In the **Computer** field, enter:
   ```
   localhost:13389
   ```
3️⃣ Click **Connect** and log in with the **Windows credentials** of the EC2 instance.

---

## ✅ Step 8: End the Session
Once you're done, **terminate the session** in AWS CLI by pressing `CTRL+C`, or manually via:
```sh
aws ssm terminate-session --session-id <SESSION_ID>
```
You can get the active **session ID** using:
```sh
aws ssm describe-sessions --state Active
```

---

## 🎯 Summary
✔ **No need to open port 3389 to the internet.**  
✔ **Secure authentication via IAM roles.**  
✔ **Easy to use and automate with AWS CLI.**  
✔ **Works without requiring a VPN or Bastion Host.**  

---

## 🔍 Troubleshooting
❌ **Issue:** `Target Not Connected`
- Ensure the EC2 instance is **running** and has **SSM Agent installed**.
- Ensure the **IAM role** is correctly attached.
- Check the **Security Group** (ensure outbound traffic allows SSM communication).

❌ **Issue:** `AccessDenied`
- Ensure your IAM user/role has `ssm:StartSession` permission.
- Verify AWS CLI credentials (`aws configure`).

❌ **Issue:** Cannot authenticate to RDP
- Use **EC2 user credentials** (Administrator, or a configured user).
- Reset the password if needed via AWS EC2 Console.

---

## 🚀 Next Steps
- Automate the process using **AWS SSM Session Manager + Terraform**.
- Integrate **SSM Port Forwarding** into CI/CD workflows.
- Enhance security with **AWS IAM Session Policies**.

---

By following this guide, you can securely connect to Windows EC2 instances using RDP over AWS SSM, eliminating the need for publicly exposed RDP ports! 🚀

