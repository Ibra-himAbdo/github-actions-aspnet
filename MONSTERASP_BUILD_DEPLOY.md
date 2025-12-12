# Build & Deploy .NET App to MonsterASP.NET using GitHub Actions

This guide explains how to automatically **Build and Deploy** your ASP.NET application to **MonsterASP.NET** using GitHub Actions.

This workflow:
- Builds your .NET project (any runtime version)
- Publishes it for **win-x86** or **win-x64** depending on your MonsterASP.NET plan
- Checks if your API is online before deployment
- Uploads the published files to MonsterASP.NET via FTP
- Cleans the server directory before deployment (optional but recommended)

---

## ✅ Requirements
Before using this workflow, make sure you have:

### 1. MonsterASP.NET FTP credentials  
Save them as GitHub Secrets:
- `FTP_HOST`
- `FTP_LOGIN`
- `FTP_PASSWORD`

### 2. Your API health URL (optional but recommended)  
Add this as GitHub Variable:

`API_HEALTH_URL`

### 3. Correct .NET Runtime  
For MonsterASP.NET:
- Free plans → **win-x86**
- Paid plans → **win-x64**

---

## 📦 Full Build & Deploy Workflow (YAML)

```yaml
name: Build and Deploy

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

concurrency:
  group: build-and-deploy-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v6

      - name: Setup .NET SDK
        uses: actions/setup-dotnet@v5
        with:
          dotnet-version: "9.0.x"   # Change to any version you need

      - name: Verify .NET version
        run: dotnet --version

      - name: Restore dependencies
        run: |
          dotnet restore <PATH_TO_MAIN_APP_CSPROJ>
          # Example:
          # dotnet restore src/Application/Application.csproj

      - name: Publish Application (win-x86, framework-dependent)
        run: dotnet publish <PATH_TO_MAIN_APP_CSPROJ> -c Release -r win-x86 --self-contained false --no-restore -o ./publish --verbosity minimal

          # Example:
          # dotnet publish src/Application/Application.csproj -c Release -r win-x86 --self-contained false --no-restore -o ./publish

      - name: List published files
        run: |
          echo "Files ready for deployment:"
          ls -la ./publish
          echo "Total files:"
          find ./publish -type f | wc -l
      
      - name: Check if API is running
        if: ${{ vars.API_HEALTH_URL != '' }}
        run: |
          echo "Checking API health at: ${{ vars.API_HEALTH_URL }}"
          STATUS=$(curl -o /dev/null -s -w "%{http_code}" "${{ vars.API_HEALTH_URL }}")
          echo "API status code: $STATUS"

          if [ "$STATUS" -eq 200 ]; then
            echo "API is running. Deployment blocked. Please stop the server first!"
            exit 1
          fi

          echo "API is offline. Safe to continue deployment."

      - name: API health check skipped (no API_HEALTH_URL variable)
        if: ${{ vars.API_HEALTH_URL == ''}}
        run: |
          echo "Skipping API health check: vars.API_HEALTH_URL is not set."

      - name: Deploy to FTP (MonsterASP.NET)
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        uses: SamKirkland/FTP-Deploy-Action@v4.3.6
        with:
          server: ${{ secrets.FTP_HOST }}
          username: ${{ secrets.FTP_LOGIN }}
          password: ${{ secrets.FTP_PASSWORD }}
          port: 21
          protocol: ftp
          local-dir: ./publish/
          server-dir: /wwwroot/
          dangerous-clean-slate: true # ⚠️ CAUTION: This deletes ALL files in the server directory before uploading new ones. Use only if you are sure you want a full replacement!
          exclude: |
            **/.git*
            **/.git*/**
            **/.github*
            **/.github*/**
