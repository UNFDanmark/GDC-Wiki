# Setup af ny windows installation

Dette dokument indeholder PowerShell-scripts til at opsætte en ny bruger, installere nødvendige programmer og konfigurere Git for GDC-deltagere.

## Oprettelse af ny bruger

Dette script opretter en ny standardbruger med navnet "Deltager" uden administratorrettigheder og uden kodeord.

```Bash
# Create new user account without admin privileges and no password
try {
    $username = "Deltager"
    New-LocalUser -Name $username -FullName "Deltager" -Description "Standard user for GDC"

    # Add the new user to the 'Users' group (standard user, not admin)
    Add-LocalGroupMember -Group "Brugere" -Member $username

    Write-Host "User $username created and configured successfully."
} catch {
    Write-Host "Error creating user: $_"
}
```
{collapsible="true" collapsed-title="create-limited-user.ps1"}

## Installér nødvendige programmer

Dette script installerer de programmer, der er nødvendige for GDC ved brug af winget.

```Bash
# Function to install applications
function Install-App($appName, $appSource) {
    try {
        $listApp = winget list --exact -q $appName
        if (![String]::Join("", $listApp).Contains($appName)) {
            Write-Host "Installing: $appName"
            if ($appSource -ne $null) {
                winget install --exact --silent $appName --source $appSource --accept-package-agreements
            } else {
                winget install --exact --silent $appName --accept-package-agreements
            }
        } else {
            Write-Host "Skipping Install of $appName"
        }
    } catch {
        Write-Host "Error installing $appName: $_"
    }
}

# List of apps to install
$apps = @(
    @{name = "Unity.Unity.2022"}, 
    @{name = "Unity.UnityHub"}, 
    @{name = "JetBrains.Rider"}, 
    @{name = "BlenderFoundation.Blender"}, 
    @{name = "KDE.Krita"}, 
    @{name = "Audacity.Audacity"}, 
    @{name = "Git.Git"}, 
    @{name = "GitHub.GitHubDesktop"},
    @{name = "GitHub.cli"},
    @{name = "BrunoBanelli.PCI-Z"},
    @{name = "Google.Chrome"}
)

# Install new apps
foreach ($app in $apps) {
    Install-App -appName $app.name -appSource $null
}
```

{collapsible="true" collapsed-title="install-apps.ps1"}

## Git Setup

Dette script opsætter Git for deltageren ved at klone nødvendige repositories og konfigurere Git-brugeren.
```Bash
# Define constants
$year = "2024"
$githubAccountName = "GDC-Teknisk"
$githubKey = "ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" # Generate a new key each year

# Prompt for group number
Write-Host -NoNewline "Enter group number: " 
$key_pressed = ([System.Management.Automation.Host.KeyInfo]$Host.UI.RawUI.ReadKey([System.Management.Automation.Host.ReadKeyOptions]::IncludeKeyDown)).Character;
Write-Host ""

# Clone repositories
try {
    Set-Location C:\Users\GDC\Desktop
    $git_repo = "https://github.com/UNFDanmark/GDC$year-GR" + $key_pressed
    git clone $git_repo
    git clone "https://github.com/UNFDanmark/GDC$year-TeachingProgramming"
    Write-Host "Repositories cloned successfully"
} catch {
    Write-Host "Error cloning repositories: $_"
}

# Configure git user
$mail = "teknisk+GDC$year-GR" + $key_pressed + "@game.unf.dk"
try {
    git config --global user.email $mail
    git config --global user.name "Deltager"
    git credential-manager configure
    git config --global url."https://$githubAccountName:$githubKey@github.com".insteadOf "https://github.com"
    Write-Host "Git user configured successfully"
} catch {
    Write-Host "Error configuring git: $_"
}

```
{collapsible="true" collapsed-title="setup-git.ps1"}