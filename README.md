# Telegram iOS Source Code Setup and Build Guide

## Overview

This guide provides comprehensive instructions for setting up, building, and distributing a custom Telegram iOS client. The project uses Bazel as its build system and requires several configuration steps.

## Step 1: Create Telegram Application

1. Visit https://core.telegram.org/api/obtaining_api_id
2. Create a Telegram application
3. Obtain your `api_id` and `api_hash`
4. Generate a random identifier for your bundle ID:
   ```bash
   openssl rand -hex 8
   ```

## Step 2: Set Up Apple Developer Account

1. Create a free or paid Apple Developer Account at https://developer.apple.com
2. Locate your:
   - Developer email address
   - Team ID (in Membership Details section)

## Step 3: Clone the Repository

Clone the Telegram iOS repository with all submodules:
```bash
git clone --recursive -j8 https://github.com/TelegramMessenger/Telegram-iOS.git
```

## Step 4: Configure Development Environment

Edit `build-system/template_minimal_development_configuration.json`:
- Add `api_id` and `api_hash`
- Include your Apple Developer email
- Add your Team ID
- Set bundle identifier as `org.{random identifier}`

## Step 5: Generate Xcode Project

Run the project generation command:
```bash
python3 build-system/Make/Make.py \
    --overrideXcodeVersion \
    --cacheDir="$HOME/telegram-bazel-cache" \
    generateProject \
    --configurationPath=build-system/template_minimal_development_configuration.json \
    --disableProvisioningProfiles \
    --xcodeManagedCodesigning
```

## Step 6: Build and Run

1. Open the generated Xcode project
2. Select an iOS simulator
3. Build and run the project (⌘R)

## Preparing for App Store Distribution

### Create App Store Connect Record

1. Go to https://appstoreconnect.apple.com
2. Click "My Apps" and "+"
3. Select "New App"
4. Fill in details:
   - Platform: iOS
   - Name: [Your App Name]
   - Bundle ID: org.[random identifier].Telegram
   - SKU: [random identifier]

### Generate Certificates

#### Development Certificate
1. Go to https://developer.apple.com/account/resources/certificates/list
2. Create iOS App Development certificate
3. Generate Certificate Signing Request (CSR) using Keychain Access

#### Distribution Certificate
1. Return to certificates page
2. Create "App Store and Ad Hoc" certificate
3. Upload CSR
4. Download and install certificate

## APNS (Apple Push Notification Service) Certificates

### Generating APNS Certificate

1. Go to https://developer.apple.com/account/resources/certificates/list

2. Create Apple Push Notifications service SSL Certificate:
   - Choose "Apple Push Notifications service SSL (Sandbox & Production)"
   - This single certificate works for both development and production environments

3. Certificate Generation Process:
   - Click the "+" button in the Certificates section
   - Select the Apple Push Notifications service SSL certificate
   - Follow the certificate generation wizard
   - You'll need to:
     * Generate a Certificate Signing Request (CSR) using Keychain Access
     * Upload the CSR to Apple's developer portal
     * Download the generated certificate

### Exporting Certificate to .p12 and .pem Format

1. Open Keychain Access
2. Find the downloaded APNS certificate
3. Export to .p12:
   - Right-click and select "Export"
   - Choose .p12 format
   - Set a strong password for the exported file

4. Convert .p12 to .pem:
   ```bash
   # Convert the certificate to PEM format
   openssl pkcs12 -in apns-cert.p12 -out apns-cert.pem -nodes -clcerts
   ```
   - You'll be prompted for the .p12 export password
   - The `-nodes` flag prevents encryption
   - The `-clcerts` flag extracts only client certificates

### Codesigning Folder Structure

Prepare a `my-codesigning` folder with the following exact structure:

```
my-codesigning/
├── certs/
│   ├── Public.cer
│   └── SelfSigned.p12
└── profiles/
    ├── BroadcastUpload.mobileprovision
    ├── Intents.mobileprovision
    ├── NotificationContent.mobileprovision
    ├── NotificationService.mobileprovision
    ├── Share.mobileprovision
    ├── Telegram.mobileprovision
    ├── WatchApp.mobileprovision
    ├── WatchExtension.mobileprovision
    └── Widget.mobileprovision
```

### Create App Identifiers

1. Go to Identifiers section
2. Create identifiers for:
   - Main App
   - Notification Service Extension
   - Share Extension
   - Widget
   - Watch App and Extension

### Create Provisioning Profiles

1. Go to Profiles section
2. Create profiles for each app identifier
3. Select "App Store" type
4. Download all profiles
5. Ensure exact naming as shown in the codesigning folder structure

### Build for Distribution

```bash
python3 build-system/Make/Make.py \
    --overrideXcodeVersion \
    --cacheDir="$HOME/telegram-bazel-cache" \
    build \
    --configurationPath=build-system/my-appstore-configuration.json \
    --codesigningInformationPath=build-system/my-codesigning \
    --buildNumber=100008 \
    --configuration=release_arm64
```

### Upload to TestFlight

1. Use Apple's Transporter app
2. Sign in with Apple Developer account
3. Upload generated IPA
4. Configure beta testing in App Store Connect