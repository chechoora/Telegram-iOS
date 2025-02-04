# Telegram iOS Source Code Basic Setup Guide

This guide covers the basic setup process for compiling the Telegram iOS source code using the simulator.

## Prerequisites

- Xcode installed (from App Store or https://developer.apple.com/download/applications)
- An Apple Developer Account (free tier is sufficient for simulator builds)
- Git installed
- Python 3.x installed

## Step 1: Create Your Telegram Application

1. Visit https://core.telegram.org/api/obtaining_api_id to create your Telegram application
2. Save your `api_id` and `api_hash` for later use
3. Generate a random identifier for your bundle ID:
```bash
openssl rand -hex 8
```

## Step 2: Set Up Apple Developer Account

1. Create a free Apple Developer Account at https://developer.apple.com if you don't have one
2. Visit https://developer.apple.com/account#MembershipDetailsCard to find your:
   - Developer email address
   - Team ID (shown in the Membership Details section)

## Step 3: Get the Source Code

Clone the Telegram iOS repository with all its submodules:
```bash
git clone --recursive -j8 https://github.com/TelegramMessenger/Telegram-iOS.git
```

## Step 4: Update Configuration

Edit `build-system/template_minimal_development_configuration.json` with your information:
- Add your `api_id` and `api_hash` from Step 1
- Use your Apple Developer email and Team ID from Step 2
- Set the bundle identifier as `org.{random identifier from step 1}`

## Step 5: Generate the Xcode Project

Run the following command to generate the project:
```bash
python3 build-system/Make/Make.py \
    --overrideXcodeVersion \
    --cacheDir="$HOME/telegram-bazel-cache" \
    generateProject \
    --configurationPath=build-system/template_minimal_development_configuration.json \
    --disableProvisioningProfiles \
    --xcodeManagedCodesigning
```

Key parameters explained:
- `--overrideXcodeVersion`: Bypasses strict Xcode version checking
- `--disableProvisioningProfiles`: Allows simulator-only builds without provisioning profiles
- `--xcodeManagedCodesigning`: Lets Xcode handle code signing

## Step 6: Build and Run

1. Open the generated Xcode project
2. Select an iOS simulator as your target device
3. Build and run the project (⌘R)

## Troubleshooting

If you encounter a build error mentioning "Telegram_xcodeproj: no such package" after a system restart:
1. Close Xcode
2. Re-run the project generation command from Step 5
3. Open the project again in Xcode

## Notes

- This setup is for simulator builds only
- No provisioning profiles or certificates are required
- Additional setup is needed for device builds and App Store distribution