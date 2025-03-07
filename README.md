# Winget GitHub Integration / Auto-Package Pipeline Setup Instructions

To set up a Winget automation pipeline in GitHub, you need to create a GitHub Actions workflow that utilizes the "wingetcreate" tool to generate Winget manifests, then automatically pushes them to your desired repository, leveraging GitHub secrets to securely store sensitive information like API tokens for private repositories. [^1] [^2] [^3]  

Reading: [Publishing a package using an action (contains example YAML)](https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions#publishing-a-package-using-an-action)

## Key steps

### Install WingetCreate

WingetCreate is a Windows-based command line tool, normally used on your own machine to create a package release on Winget.

If you are new to GitHub actions, find the Actions page in your repository, ignore all of the suggested items on screen, and instead click on the small blue link "set up a workflow yourself".

| ![image](https://github.com/user-attachments/assets/8c47a005-8d55-4c4e-8f8b-26c7da35db11) |
|------------------------------------------------------------------------------------------|

| ![image](https://github.com/user-attachments/assets/9dc4ed6d-722c-4733-a4d8-a914d3d994c5) |
|-------------------------------------------------------------------------------------------|

From here, follow the example in the Reading above to create a script.

I recommend creating a git branch called "release" in your project. Whenever you copy your files from main, or your working branch, to this branch, it will be inferred as issuing a release of your software.

This script uses `run` commands to execute `wingetcreate`. The appropriate documentation for this feature is here: [Workflow Syntax for Github actions - Run command](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsrun)

If you already use GitHub actions, in your GitHub Actions workflow, add a step to install the "wingetcreate" tool using the appropriate package manager for your chosen operating system. [^1] [^3] [^4]

### Create a Manifest File

Within your repository, create a manifest file containing the necessary information for your Winget package (name, publisher, version, etc.). [^3] [^4]  For example, UWP apps would use a .appxmanifest file.

Warning: I do not know if an app manifest file is required, or if there is a way to do whatever it does, but manually.

Reading:

[Using Windows Package Manager Manifest Creator in a CI/CD pipeline](https://github.com/microsoft/winget-create?tab=readme-ov-file#using-windows-package-manager-manifest-creator-in-a-cicd-pipeline)

[Example YAML from DevHome project](https://github.com/microsoft/devhome/blob/main/.github/workflows/winget-submission.yml)


### Generate Winget Manifest

Use a GitHub Actions step with the "wingetcreate" command to generate the Winget manifest file from your other manifest file, specifying any required options like the target repository URL. [^1] [^2] [^4]  

### Authentication for Private Repositories

Create a GitHub secret to store your personal access token (PAT) needed to push to a private repository. [^2] [^4]  

### Push to Repository:

Add a step in your workflow to push the generated Winget manifest to your desired repository using the stored PAT. [^2] [^3] [^4]  

Example GitHub Actions Workflow:

```
name: Create and Publish a Package on Winget


on:
  push: 
    branches: ['release']

env:
  IMAGE_NAME: ${{ github.repository }} # You can replace this variable with a string if the repository is not a great name.
     # IMAGE_NAME: ${{ github.repository }} resolves to the repository's owner and name, separated by a forward slash: your-username/your-repo-name
     # custom replacement might be like AMDphreak/really-cool-packagename
     # This image name is used to produce a package named "AMDphreak.really-cool-packagename" on the Winget repository.

jobs:
  build-and-push-image:
    runs-on: windows-latest

    permissions:
      contents: read
      packages: write
      attestations: write
      id-token: write
      # Sets the permissions granted to the GITHUB_TOKEN

    steps:
      - name: Checkout repository
      - uses: actions/checkout@v4

      - name: Install WingetCreate
        run: |
          winget install wingetcreate
          # Install WingetCreate using your preferred method (e.g., Chocolatey, Scoop)
          # The | character allows typing multiple commands.

      - name: Generate Winget Manifest
        run: | 
          wingetcreate --appxmanifest "path/to/your/appxmanifest.xml" --output "your_package_manifest.yaml" --repo "your-private-repo-url" --token ${{ secrets.GITHUB_TOKEN }}

      - name: Push to Repository
        run: | 
          git config user.name "GitHub Actions Bot"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git add your_package_manifest.yaml
          git commit -m "Update Winget Manifest"
          git push origin main
```

## Important Considerations

• Access Level: Ensure your GitHub Actions runner has the required permissions to access your private repositories and execute the necessary commands. [^2] [^4]  
• Versioning: Implement a strategy to manage package versions within your manifest files to maintain consistency across releases. [^2] [^4] [^5]  
• Testing: Set up a testing process to verify that the generated Winget packages install correctly on target systems before pushing to your repository. [^1] [^2] [^4]  

This was initially generated with Google Search and edited by hand. Generative AI is experimental.

[^1]: https://github.com/microsoft/winget-create
[^2]: https://techcommunity.microsoft.com/blog/educatordeveloperblog/wingetcreate-keeping-winget-packages-up-to-date/4037598
[^3]: https://github.com/microsoft/winget-create/blob/main/pipelines/azure-pipelines.release.yml
[^4]: https://learn.microsoft.com/en-us/shows/open-at-microsoft/wingetcreate-keeping-winget-packages-up-to-date
[^5]: https://github.com/marketplace/actions/install-winget[-] https://zenn.dev/akari0519/articles/50d7922ecc69ec
