# Sleep Settings Templates

This directory contains domain-style templates for configuring sleep and lock settings on Windows devices managed by Microsoft Intune.

## Overview

These templates provide standardized power management and security configurations that can be deployed to Windows hosts through Intune device configuration policies. Each template is designed for different use cases and security requirements.

## Available Templates

### 1. Baseline Sleep Timeout
**File:** `Baseline-Sleep-Timeout.json`

**Configuration:**
- Black screen saver: 3 minutes
- Device lock timeout: 15 minutes
- Domain reauthentication required at Windows logon

**Use Case:** Standard workstations in office environments where moderate security and power management are required.

**Settings:**
- Screen timeout (AC/Battery): 180 seconds (3 minutes)
- Lock timeout: 15 minutes
- Screen saver: Enabled with blank/black screen
- Domain authentication: Required after lock

---

### 2. Laptop Timeout Variant
**File:** `Laptop-Timeout-Variant.json`

**Configuration:**
- Black screen saver: 2 minutes
- Device lock timeout: 5 minutes

**Use Case:** Mobile devices and laptops where faster security lockout is needed due to portability and increased risk of unauthorized access.

**Settings:**
- Screen timeout (AC/Battery): 120 seconds (2 minutes)
- Lock timeout: 5 minutes
- Screen saver: Enabled with blank/black screen

---

### 3. 4-Hour Lock Variant
**File:** `4Hour-Lock-Variant.json`

**Configuration:**
- Device lock timeout: 4 hours (240 minutes)
- Screen saver: Disabled
- Extended screen timeout: 4 hours

**Use Case:** Workstations used for long-running tasks, server consoles, or environments where frequent re-authentication would disrupt workflow, but security still requires eventual lockout.

**Settings:**
- Lock timeout: 240 minutes (4 hours)
- Screen saver: Disabled
- Screen timeout (AC/Battery): 14400 seconds (4 hours)

---

## Deployment Instructions

### Prerequisites
- Microsoft Intune subscription
- Azure AD tenant
- Devices enrolled in Intune
- PSIntuneAuth PowerShell module (for script-based deployment)

### Method 1: Using Import Script

1. Use the `Import-IntuneDeviceConfigurationProfile.ps1` script from the parent directory:

```powershell
.\Import-IntuneDeviceConfigurationProfile.ps1 `
    -TenantName "yourtenant.onmicrosoft.com" `
    -Platform "Windows" `
    -Path ".\Sleep Settings Templates" `
    -Verbose
```

2. After import, assign the policy to appropriate device groups in the Intune portal.

### Method 2: Manual Import via Intune Portal

1. Sign in to the [Microsoft Endpoint Manager admin center](https://endpoint.microsoft.com/)
2. Navigate to **Devices** > **Configuration profiles**
3. Click **Create profile**
4. Select:
   - Platform: **Windows 10 and later**
   - Profile type: **Templates** > **Custom**
5. Click **Create**
6. Copy the contents of the desired JSON file and paste into the configuration
7. Assign to appropriate Azure AD groups

### Method 3: Using Microsoft Graph API

You can also deploy these configurations using the Microsoft Graph API:

```powershell
$uri = "https://graph.microsoft.com/v1.0/deviceManagement/deviceConfigurations"
$body = Get-Content ".\Baseline-Sleep-Timeout.json" -Raw
Invoke-RestMethod -Uri $uri -Headers $authHeader -Method Post -Body $body -ContentType "application/json"
```

## Configuration Details

### Policy Settings Explained

#### Screen Timeout Settings
- **DisplayOffTimeoutPluggedIn**: Controls when the screen turns off when device is plugged into AC power
- **DisplayOffTimeoutOnBattery**: Controls when the screen turns off when device is on battery power
- Values are in seconds

#### Lock Settings
- **MaxInactivityTimeDeviceLock**: Controls when the device automatically locks after inactivity
- Values are in minutes for this setting

#### Screen Saver Settings
- **ScreenSaverActive**: Enables (1) or disables (0) the screen saver
- **ScreenSaverTimeout**: Time in seconds before screen saver activates
- **ScreenSaverIsSecure**: Requires password (1) when resuming from screen saver (uses blank screen)

## Customization

To customize these templates for your organization:

1. Copy the desired template JSON file
2. Modify the values as needed:
   - Update `displayName` to match your naming convention
   - Adjust timeout values (remember: screen timeouts are in seconds, lock timeout is in minutes)
   - Add or remove settings as required
3. Update the `description` field to document your changes
4. Deploy using one of the methods above

## Important Notes

- **Minutes vs Seconds**: Be aware that different settings use different time units:
  - Screen timeouts: **seconds**
  - Lock timeout: **minutes**
  
- **Policy Conflicts**: These policies will override user-configured power settings. Ensure this aligns with your organization's policies.

- **Domain Authentication**: The baseline template includes domain reauthentication requirements. Ensure your environment supports this (requires domain-joined or hybrid Azure AD joined devices).

- **Testing**: Always test configurations on a pilot group before deploying to production devices.

## Support

For issues or questions:
- Review the Import/Export scripts in the parent directory
- Check Microsoft Intune documentation for policy settings
- Verify that the devices meet the requirements for Intune management

## Related Resources

- [Microsoft Intune Device Configuration](https://docs.microsoft.com/en-us/mem/intune/configuration/)
- [Windows Power Management Settings](https://docs.microsoft.com/en-us/windows/client-management/mdm/policy-csp-power)
- [Device Lock Policy CSP](https://docs.microsoft.com/en-us/windows/client-management/mdm/policy-csp-devicelock)

## Version History

- **v1.0** - Initial release with three template variants
  - Baseline Sleep Timeout (3min screen, 15min lock)
  - Laptop Timeout Variant (2min screen, 5min lock)
  - 4-Hour Lock Variant (4hr lock, no screensaver)
