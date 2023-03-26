# Automated .NET Library Build and Publishing to GitHub Packages

## Prerequisites

- A GitHub repository containing your .NET library project
- A GitHub Personal Access Token (PAT) with the `write:packages` and `read:packages` scopes

## Instructions

1. **Set up continuous integration with GitHub Actions**:
   - In your GitHub repository, create a new directory named `.github` at the root level, and inside the `.github` directory, create another directory called `workflows`.
   - In the `workflows` directory, create a new YAML file, such as `build-dotnet-library.yml`.

2. **Write the build script**:
   - In the `build-dotnet-library.yml` file, define the workflow for building your .NET library. Here's an example:

```yaml
name: Build .NET Library

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: windows-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: '6.0.x' # Adjust the .NET version as needed

    - name: Restore dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --configuration Release --no-restore

    - name: Test
      run: dotnet test --no-restore --verbosity normal

    - name: Publish
      run: dotnet publish --configuration Release --no-build --output ./artifacts

    - name: Upload artifacts
      uses: actions/upload-artifact@v2
      with:
        name: library-artifacts
        path: ./artifacts/
```

3. **Configure the CI pipeline to run the build script**:
   - Push the `build-dotnet-library.yml` file to your repository. GitHub Actions should automatically detect the new workflow and start running it whenever there's a push or pull request to the `main` branch.

4. **Add the Personal Access Token (PAT) to your repository**:
   - In your GitHub repository, go to "Settings" > "Secrets" and click on "New repository secret".
   - Add the PAT as a new secret named `GH_PACKAGES_TOKEN`.

5. **Update the build script to publish the library to GitHub Packages**:
   - Replace the "Publish to NuGet" step in the `build-dotnet-library.yml` workflow file with the following step to publish to GitHub Packages:

```yaml
    - name: Publish to GitHub Packages
      run: |
        dotnet nuget add source https://nuget.pkg.github.com/${{ github.repository_owner }}/index.json --name github --username ${{ github.repository_owner }} --password ${{ secrets.GH_PACKAGES_TOKEN }} --store-password-in-clear-text
        dotnet nuget push ./artifacts/*.nupkg --source github
      if: github.event_name == 'push' && github.ref == 'refs/heads/main'
```

6. **Update the .NET library project's configuration**:
   - In your .NET library project's `.csproj` file, add the following `RepositoryUrl` and `PackagePublishUrl` properties:

```xml
<PropertyGroup>
  <RepositoryUrl>https://github.com/yourusername/yourrepository</RepositoryUrl>
  <PackagePublishUrl>https://nuget.pkg.github.com/yourusername</PackagePublishUrl>
</PropertyGroup>
```

Replace `yourusername` and `yourrepository` with your GitHub username and repository name.

7. **Create a `nuget.config` file in your .NET library project**:
   - In the root directory of your .NET library project, create a new file called `nuget.config` with the following content:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="github" value="https://nuget.pkg.github.com/yourusername/index.json" />
  </packageSources>
</configuration>
```

Replace `yourusername` with your GitHub username or organization name.

**Commit and push your changes**:
   - Commit and push the updated workflow file, project configuration, and `nuget.config` to your repository. GitHub Actions should automatically detect the new workflow and start running it whenever there's a push or pull request to the `main` branch. The library will be published to GitHub Packages whenever there's a push to the `main` branch.

