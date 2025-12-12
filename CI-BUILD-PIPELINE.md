# **.NET Build Pipeline**


## 📘 Overview
This pipeline builds a .NET project using **any .NET version** you specify.  
Its main purpose is to ensure:

- The project compiles successfully  
- All NuGet dependencies are restored  
- The build runs in Release mode  
- Build errors are caught early during CI  

It is triggered automatically on:

- Push to the **main** branch  
- Pull Requests targeting **main**

---

## 🧩 YAML File (build.yml)

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
        
      - name: Setup .NET SDK
        uses: actions/setup-dotnet@v5
        with:
          dotnet-version: '9.0.x'   # Change this to any version (7.0.x, 8.0.x, 10.0.x...)

      - name: Verify .NET version
        run: dotnet --version

      - name: Restore dependencies
        run: dotnet restore src/Application/Application.csproj

      - name: Build
        run: dotnet build src/Application/Application.csproj --configuration Release --no-restore --verbosity minimal
