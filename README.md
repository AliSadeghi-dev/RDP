# Free Windows VPS with RDP Access




This  guide explains how to set up a **free Windows VPS** using **GitHub Actions** with  **ngrok** for RDP access. Follow the steps below to replicate the workflow.

---

## **Features**

- Free Windows VPS using GitHub's infrastructure.
- Secure RDP connection using **ngrok**.
- Fully automated setup via **GitHub Actions**.

---

## **GitHub Actions Free Usage Limits**

GitHub provides **free CI/CD minutes** for workflows, which includes this setup:

- **Public repositories**:
  - Unlimited free minutes for GitHub Actions workflows.
- **Private repositories**:
  - **Free Tier**:
    - 2,000 minutes per month for **Free and Pro plans**.
    - 3,000 minutes per month for **Team plans**.
    - 50,000 minutes per month for **Enterprise plans**.
  - Usage above these limits incurs additional charges.

For more details, refer to [GitHub Actions Pricing](https://github.com/features/actions#pricing-details).

---

## **Prerequisites**

1. A **GitHub account**.
2. A **ngrok account** to obtain an authtoken.
3. An **RDP client** (e.g., **Remote Desktop Connection**).

---

## **Setup Instructions**

### Step 1: Fork or Clone the Repository

1. Open your GitHub account.
2. Create a new repository or fork an existing one.
3. Clone the repository locally (optional) using:

   ```bash
   git clone https://github.com/<username>/<repo-name>.git
   cd <repo-name>
   ```

### Step 2: Retrieve Your ngrok Authtoken

1- Log in to your ngrok dashboard at https://dashboard.ngrok.com.

2- Go to the Setup & Installation section.

3- Copy your ngrok authtoken, which is a long alphanumeric string.

### Step 3: Add ngrok Authtoken as a GitHub Actions Secret

1- Go to your GitHub repository.

2- Navigate to Settings > Secrets and Variables > Actions.

3- Click New repository secret.

4- Enter the following details:

- **Name:** `NGROK_AUTH_TOKEN`
- **Secret:** Paste your ngrok authtoken.

5- Click Add secret.

### Step 4: Create the GitHub Actions Workflow

1- Go to the Actions tab in your repository.

2- Click on Set up a workflow yourself or New workflow.

3- Replace the default content with the following script and save it as .github/workflows/main.yml:


```
name: CI-RDP

on: [push, workflow_dispatch]

jobs:
  build:
    runs-on: windows-latest
    steps:
    - name: Download
      run: Invoke-WebRequest https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-windows-amd64.zip -OutFile ngrok.zip
    - name: Extract
      run: Expand-Archive ngrok.zip
    - name: Auth
      run: .\ngrok\ngrok.exe authtoken $Env:NGROK_AUTH_TOKEN
      env:
        NGROK_AUTH_TOKEN: ${{ secrets.NGROK_AUTH_TOKEN }}
    - name: Enable TS
      run: Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0
    - run: Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
    - run: Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1
    - run: Set-LocalUser -Name "runneradmin" -Password (ConvertTo-SecureString -AsPlainText "P@ssw0rd!" -Force)
    - name: Create Tunnel
      run: .\ngrok\ngrok.exe tcp 3389
```


4- Commit and push the changes to your repository.

### Step 5: Run the Workflow

1- Go to the Actions tab in your repository.

2- Select the `CI-RDP` workflow.

3- Click on Run workflow and wait for it to complete.

### Step 6: Retrieve the TCP URL from the Logs

1- After the workflow finishes, expand the Create Tunnel step in the workflow logs.

2- Locate the ngrok forwarding URL. It will look like this:

Forwarding `tcp://2.tcp.ngrok.io:12828` -> `127.0.0.1:3389`


3- Copy the part after `tcp://` (e.g., `2.tcp.ngrok.io:12828`).

### Step 7: Connect via RDP

1- Open your **RDP client** (e.g., **Remote Desktop Connection** on Windows).

2- Enter the following credentials:

- **Host:** The TCP URL you retrieved (e.g., `2.tcp.ngrok.io:12828`).
- **Username:**
  ```
  runneradmin
  ```
- **Password:**
  ```
  P@ssw0rd!
  ```

3- Click Connect.



## **Important Notes**
	
**1. Dynamic Tunnel URL:**

- The ngrok TCP URL changes for each session. Ensure you copy the correct URL from the workflow logs for each new session.

**2. Workflow Expiration:**

- The VPS will automatically stop after the GitHub workflow run ends (6-hour limit). To restart, rerun the workflow.

**3. Keep the CMD Window in the VPS Open:**

- After logging into the VPS, do not close the CMD window running ngrok. Closing this window will terminate the tunnel and disconnect your RDP session.

**4. ngrok Free Tier Limitations:**

- Free accounts are limited to 40 connections per minute.
- The TCP URL generated by ngrok is ephemeral and will change each time you run the workflow.

**5. Port Information:**

- The workflow uses port 3389 for RDP by default. Ensure this port is open and not blocked by your local firewall or ISP.

**6. Optional Persistent Tunnel:**

- If you need a persistent TCP URL, upgrade to a paid ngrok plan and configure a static TCP endpoint in your ngrok dashboard.



## **Troubleshooting Tips**

**1. RDP Connection Fails:**
- Double-check the TCP URL and ensure you’re using the correct URL from the workflow logs.
- Ensure port 3389 is not blocked by your local network or firewall.

**2. Workflow Errors:**
- Verify that the `NGROK_AUTH_TOKEN` is correctly added as a secret.
- Check the workflow logs for error messages during execution.

**3. ngrok Tunnel Disconnects:**
- Make sure the CMD window inside the VPS **remains open**.
- If the tunnel disconnects frequently, consider upgrading to a paid ngrok plan.


## **Customization**
- To change the RDP password, modify the following line in main.yml:

```
- run: Set-LocalUser -Name "runneradmin" -Password (ConvertTo-SecureString -AsPlainText "YourNewPassword" -Force)
```


## **Limitations**

**1. Temporary Access:** The VPS is active only while the GitHub Actions workflow is running.

**2. Security:** Use only for testing and development; avoid exposing sensitive information.

**3. Resource Limits:** The instance uses GitHub’s free runner resources (2-core CPU, 7GB RAM).


## **Disclaimer**

This setup is intended for educational and development purposes only. Ensure compliance with GitHub and ngrok terms of service. The authors are not responsible for misuse or unintended consequences.

**Enjoy your free Windows VPS!**

