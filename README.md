Setup Ngrok and RDP with GitHub Actions

This repository contains a GitHub Actions workflow that automates the following tasks on a windows-latest runner:

    Downloads Ngrok: The latest version of Ngrok is downloaded and extracted.
    Authenticates Ngrok: The Ngrok authtoken is used to authenticate the account for creating secure tunnels.
    Configures Remote Desktop (RDP): The workflow enables Remote Desktop connections and adjusts necessary registry settings for RDP.
    Sets up the runneradmin user: It creates or updates the runneradmin local user account with a secure password.

Workflow Steps

The workflow is automatically triggered on every push or can be manually triggered using workflow_dispatch.
1. Download Ngrok

    Downloads the latest stable version of Ngrok for Windows.
    The download link points to a version hosted on Ngrok's distribution platform.

2. Extract Ngrok

    The downloaded Ngrok ZIP archive is extracted into a folder to prepare for usage.

3. Authenticate Ngrok

    Uses the NGROK_AUTH_TOKEN secret to authenticate the Ngrok client. This allows you to establish secure tunnels.

4. Configure Remote Desktop

    The registry keys are modified to:
        Enable Terminal Server connections (RDP).
        Enable the "UserAuthentication" setting for RDP to ensure proper authentication for remote connections.
    Windows Firewall rules are modified to allow Remote Desktop traffic.

5. Set up runneradmin Account

    The runneradmin local user account password is set using the Set-LocalUser cmdlet.
    The password is securely passed using ConvertTo-SecureString to ensure it is handled safely.

6. Create Ngrok Tunnel

    Finally, a secure TCP tunnel to port 3389 is created using Ngrok, allowing access to the Remote Desktop (RDP) port from anywhere.

Requirements

To use this workflow, you need to set up the following secrets in your GitHub repository:

    NGROK_AUTH_TOKEN: Your Ngrok authentication token (you can find this after signing up on the Ngrok website).
    RUNNER_ADMIN_PASSWORD: The password you want to assign to the runneradmin account. This is stored securely as a GitHub secret.

Example GitHub Actions Workflow File

name: Cl

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

Additional Information

    Ngrok Tunnel: Once the tunnel is created, it will provide a public URL that can be used to access your Windows RDP session through port 3389. This allows remote desktop access over the internet securely via Ngrok's tunnel.
    Password: The runneradmin password is configured in the workflow as "P@ssw0rd!", but it is recommended to use a more secure and dynamic password using GitHub Secrets.
    Firewall & Registry: Ensure the firewall and registry settings are properly configured to allow RDP if you are using this workflow on a production or real environment.

Troubleshooting

    Ngrok Tunnel Failures: If the tunnel doesn't start, check the logs for errors related to Ngrok authentication or network issues.
    Firewall or Registry Configuration: If RDP doesn't work, ensure that the registry keys were updated correctly and that the firewall rule for RDP is active.
