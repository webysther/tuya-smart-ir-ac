# Tuya Cloud setup

The integration talks to your devices through the **Tuya IoT Development Platform** (the "Tuya Cloud"). Before you add anything in Home Assistant, you need a cloud project that:

1. Is created in the **same data center** as your Smart Life / Tuya Smart app account.
2. Has the required **API services** authorized.
3. Is **linked to your app account**, so it can see your devices.
4. *(Optional)* Has the **Message Service** enabled for real-time updates (Tuya Pulsar).

← Back to [README](../README.md)

---

## 1. Create a cloud project

1. Log in to the [Tuya IoT Development Platform](https://platform.tuya.com).
2. In the left sidebar, go to **Cloud → Project Management** (or **Cloud → Development**).
3. Click **Create Cloud Project** (or open an existing one).
4. Fill in the form:
   - **Project Name / Description**: anything you like.
   - **Industry**: `Smart Home`.
   - **Development Method**: `Smart Home`.
   - **Data Center**: you **must** pick the data center of your app account. See [Choosing the data center](#choosing-the-data-center).
5. Click **Create**.
6. In the **Authorize API Services** wizard, make sure **IoT Core** and **IR Control Hub Open Service** are selected, then click **Authorize**.

<!-- screenshot: images/tuya-create-project.png — "Create Cloud Project" dialog (English UI) -->

### Choosing the data center

The integration supports only these Tuya data centers. Pick the one that matches your app account, both in the Tuya project **and** in Home Assistant:

| Option in Home Assistant | Value | Tuya data center | API endpoint |
|--------------------------|-------|------------------|--------------|
| Europe | `eu` | Central Europe | `openapi.tuyaeu.com` |
| America | `us` | Western America | `openapi.tuyaus.com` |
| India | `in` | India | `openapi.tuyain.com` |
| China | `cn` | China | `openapi.tuyacn.com` |
| Singapore | `sg` | Singapore | `openapi-sg.iotbing.com` |

To find which data center your app account uses, check the [Tuya data center mapping](https://developer.tuya.com/en/docs/iot/oem-app-data-center-distributed?id=Kafi0ku9l07qb). In the Smart Life app, the region is the country you selected when you created the account (**Me → Settings → Account and Security → Region**).

> ⚠️ If the data centers do not match, Home Assistant shows **"Authentication failed: Invalid data center."** and the project shows no devices.

## 2. Link your Smart Life / Tuya Smart app account

1. Open your project and go to the **Devices** tab.
2. Select **Link App Account** (do **not** use "Link My App").
3. Click **Add App Account**. A QR code appears.
4. In the Smart Life / Tuya Smart app, tap **[+]** or the scan icon (top right) and scan the QR code.
5. Confirm the authorization on your phone.

Your devices are now listed under **Devices → All Devices**.

## 3. Collect credentials and IDs

### Credentials

On the project **Overview** tab, copy:

- **Access ID / Client ID** → used as **Tuya access ID**
- **Access Secret / Client Secret** → used as **Tuya access secret**

### Device IDs

You will need different IDs depending on what you are adding:

| What you add | ID(s) required |
|--------------|----------------|
| Air conditioner | **Infrared hub ID** (the physical IR blaster) + **Air conditioner ID** (the AC remote created under the blaster) |
| Generic device | **Infrared hub ID** + **Device ID** (the remote created under the blaster) |
| Temperature/humidity sensor | **Device ID** of the sensor |

You can find these IDs in two places:

- **Tuya IoT Platform**: project → **Devices → All Devices**. The *Device ID* column lists the IR blaster and every remote created under it.
- **Smart Life app**: open the device (or remote) → ✏️ (edit, top right) → **Device Information** → **Virtual ID**.

> 💡 The "infrared hub" is the **physical IR blaster**. The AC and other appliances are **remotes** (virtual devices) that you created inside the blaster in the app. Each one has its own ID.

## 4. *(Optional)* Enable the Message Service (Tuya Pulsar)

Without this step, Home Assistant only learns about changes made **outside Home Assistant** (for example in the Smart Life app) at the next poll, which happens every 300 s by default. With Pulsar, they show up within seconds.

1. In the Tuya IoT Platform, go to **Cloud → Development →** *your project* **→ Message Service**.
2. Turn the **Message Service** switch **ON**.
3. Open the **Production Environment** tab.
   > ⚠️ Home Assistant connects only to the **Production** environment. Anything configured under *Test Environment* is ignored.
4. Under **Messaging Rules / Subscriptions**, enable these message types (**BizCode**):
   - `devicePropertyMessage` (Device Property Message): required for climate and sensor updates
   - `statusReport` (Status Report): required for real-time reporting
   - `deviceEventMessage` (Device Event Message)
   - `deviceActionResponseMessage` (Device Action Response Message)
5. In Home Assistant, enable **Enable Tuya Pulsar Stream** in the hub settings (**Configure → Hub Settings**).

<!-- screenshot: images/tuya-message-service.png — Message Service, Production Environment, subscriptions enabled -->

> 💡 If the message types are already enabled but nothing arrives, disable them, save, then enable them again. This forces Tuya to rebuild its routing rules.

About 60 seconds after startup, the integration checks whether any message has arrived on the stream. If none has, it creates a persistent notification called **"Tuya Pulsar Stream Inactive"** with these same instructions. See [Troubleshooting](troubleshooting.md#tuya-pulsar-stream-inactive-notification).

## 5. Keep the subscription active

The free **IoT Core** service is a trial subscription that expires. When it expires, every API call fails. Renew it under **Cloud → Cloud Services → IoT Core** (or click **Extend trial** when prompted). See [Troubleshooting](troubleshooting.md#tuya-subscription-expired).

---

Next: [Configuration reference](configuration.md)
