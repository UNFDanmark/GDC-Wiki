# Bloker webside lokalt

For at kunne køre powershell scripts på en windows computer skal det slås til ved at køre denne kommando med administrator rettigheder.

```Shell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Det følgende script har 5 dele:
1. Sti til hosts fil
2. Funktion der tilføjer domæner til hosts filen
3. Liste af domæner som skal blokkeres
4. Ip adresse som domæner skal omdirigeres til
5. Løkke som blokkerer domæner

```Shell
# Define the path to the hosts file
$hostsFilePath = "$env:SystemRoot\System32\drivers\etc\hosts"

# Function to add a domain to the hosts file
function Add-DomainToHostsFile {
    param (
        [string]$ipAddress,
        [string]$domain
    )
    
    # Check if the entry already exists
    $entry = "$ipAddress`t$domain"
    $hostsFileContent = Get-Content -Path $hostsFilePath
    if ($hostsFileContent -contains $entry) {
        Write-Output "Entry '$entry' already exists in the hosts file."
    } else {
        # Add the new entry
        try {
            Add-Content -Path $hostsFilePath -Value $entry
            Write-Output "Entry '$entry' has been added to the hosts file."
        } catch {
            Write-Error "Failed to add entry '$entry' to the hosts file. Error: $_"
        }
    }
}

# List of domains to block
$domainsToBlock = @(
    "www.roblox.com",
    "www.netflix.com"
)

# IP address to redirect the domains to (localhost)
$blockIpAddress = "127.0.0.1"

# Block each domain
foreach ($domain in $domainsToBlock) {
    Add-DomainToHostsFile -ipAddress $blockIpAddress -domain $domain
}
```
{collapsible="true" collapsed-title="block-domains.ps1"}