# UploadToAzure
Upload to Azure via FTP using Powershell Script

Simply build a powershell script, and execute to be able to deploy your files into Azure.

**N.b. if the app dosen't exist, it will be created!**

```powershell
$appdirectory="."
$webappname="WEB_APP_NAME"
$location="Central US"
$ressourceGroup="RESSOURCE_GROUP_NAME"
 
# Create a resource group.
New-AzureRmResourceGroup -Name $ressourceGroup -Location $location
 
# Create an App Service plan in `Free` tier.
New-AzureRmAppServicePlan -Name $webappname -Location $location `
-ResourceGroupName $ressourceGroup -Tier Free
 
# Create a web app.
New-AzureRmWebApp -Name $webappname -Location $location -AppServicePlan $webappname `
-ResourceGroupName $ressourceGroup
 
# Get publishing profile for the web app.
[xml]$xml = (Get-AzureRmWebAppPublishingProfile -Name $webappname `
-ResourceGroupName $ressourceGroup `
-OutputFile null)
 
# Extract connection information from publishing profile.
$username = $xml.SelectNodes("//publishProfile[@publishMethod=`"FTP`"]/@userName").value
$password = $xml.SelectNodes("//publishProfile[@publishMethod=`"FTP`"]/@userPWD").value
$url = $xml.SelectNodes("//publishProfile[@publishMethod=`"FTP`"]/@publishUrl").value
 
# Upload files recursively.
Set-Location $appdirectory
$webclient = New-Object -TypeName System.Net.WebClient
$webclient.Credentials = New-Object System.Net.NetworkCredential($username,$password)
$files = Get-ChildItem -Path $appdirectory -Recurse | Where-Object{!($_.PSIsContainer)}
foreach ($file in $files)
{
    $relativepath = (Resolve-Path -Path $file.FullName -Relative).Replace(".\", "").Replace('\', '/')
    $uri = New-Object System.Uri("$url/$relativepath")
    "Uploading to " + $uri.AbsoluteUri
    $webclient.UploadFile($uri, $file.FullName)
}
$webclient.Dispose()
```
