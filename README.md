# GitHub Actions for ASP.NET

Automate **build, test, and deployment** of your ASP.NET projects with GitHub Actions.

## Features

- Build and restore ASP.NET projects
- Publish for Windows (x86/x64)
- Optional API health check before deployment
- Clean deployment to `/wwwroot` or target directories
- Automatic deployment via FTP/SFTP
- Configurable branches and environment variables
- Explicit project path support `(src/Application/Application.csproj)`

## Getting Started

1. Create a folder in your repository: `.github/workflows`

2. Create one or more YAML files inside it (e.g., `build.yml` for simple builds, `deploy.yml` for build + deploy).  
   Each file below is a complete, ready-to-use workflow.

3. Commit and push to your repository – GitHub will automatically detect and run them.

4. For deployment workflows: Add your FTP/SFTP credentials as **GitHub Secrets** (Repository → Settings → Secrets and variables → Actions).

## Workflow: Basic Build (`build.yml`)

```yaml
name: Build

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v6
        
      - name: Setup .NET 9
        uses: actions/setup-dotnet@v5
        with:
          dotnet-version: '9.0.x'
          
      - name: Verify .NET version
        run: dotnet --version

      - name: Restore dependencies
        run: dotnet restore src/Application/Application.csproj

      - name: Build
        run: dotnet build src/Application/Application.csproj --configuration Release --no-restore --verbosity minimal
