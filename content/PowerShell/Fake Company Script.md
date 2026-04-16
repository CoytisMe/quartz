---
category: script
publish: true
tags:
  - powershell
  - script
  - sharepoint
  - claude
---
## What is it?

Literally what it says on the tin, created 100 folders of fake folder structure that looks like a finance company. Word Docs and all.

I used to demo a SharePoint library:

![[Fake Company Script.png]]



```powershell
# New-FinanceDemoFiles-100.ps1
# 100 fake finance client folders for SharePoint demos.
# Edit $root to your desired output path before running.

$root = "C:\Fakecompany"

$docxBytes = [System.Convert]::FromBase64String("UEsDBBQAAAAIAK0dh1x5bjPX6AAAAK0BAAATAAAAW0NvbnRlbnRfVHlwZXNdLnhtbH1QyU7DMBD9FWuuKHHggBCK0wPLETiUDxjZk8SqN3nc0v49Tlt6QIXjzFv1+tXeO7GjzDYGBbdtB4KCjsaGScHn+rV5AMEFg0EXAyk4EMNq6NeHRCyqNrCCuZT0KCXrmTxyGxOFiowxeyz1zJNMqDc4kbzrunupYygUSlMWDxj6Zxpx64p42df3qUcmxyCeTsQlSwGm5KzGUnG5C+ZXSnNOaKvyyOHZJr6pBJBXExbk74Cz7r0Ok60h8YG5vKGvLPkVs5Em6q2vyvZ/mys94zhaTRf94pZy1MRcF/euvSAebfjpL49zD99QSwMEFAAAAAgArR2HXJv9N+qtAAAAKQEAAAsAAABfcmVscy8ucmVsc43POw7CMAwG4KtE3mlaBoRQ0y4IqSsqB7ASN61oHkrCo7cnAwNFDIy2f3+W6/ZpZnanECdnBVRFCYysdGqyWsClP232wGJCq3B2lgQsFKFt6jPNmPJKHCcfWTZsFDCm5A+cRzmSwVg4TzZPBhcMplwGzT3KK2ri27Lc8fBpwNpknRIQOlUB6xdP/9huGCZJRydvhmz6ceIrkWUMmpKAhwuKq3e7yCzwpuarF5sXUEsDBBQAAAAIAK0dh1zp+cGTewAAAJsAAAAcAAAAd29yZC9fcmVscy9kb2N1bWVudC54bWwucmVsc1XMQQ4CIQyF4auQ7h3QhTEGmJ0HMHqAZqYCkSmEEqO3l6UuX/68z87vLasXNUmFHewnA4p4KWvi4OB+u+xOoKQjr5gLk4MPCczeXiljHxeJqYoaBouD2Hs9ay1LpA1lKpV4lEdpG/YxW9AVlycG0gdjjrr9GuCt/kP9F1BLAwQUAAAACACtHYdceKeoNPMAAAB4AQAAEQAAAHdvcmQvZG9jdW1lbnQueG1sbZDBTkMhEEV/ZcJaH9WFMU1fuzGuXJioH0BhbCcBBpnBZ/9eaE1MjJtLZmDOnctm95UifGIV4jybm2llALPnQPkwm7fXx+t7A6IuBxc542xOKGa33SzrwL4lzAodkGW9zOaoWtbWij9icjJxwdzv3rkmp72sB7twDaWyR5HOT9HerlZ3NjnKZiD3HE7jLEPqEN0+/Nhs7KiG1rOWvw+fuGICKtISBI5cQUjBJdQr8JwFvaK2Ci5QIfHdHzCSTvCCoQ8AUpPEARRT6cOUPQUKrQdsCtHtOx5QL2iE5A7ZgYv00dz0727D8Lnac+MSzP5+2vYbUEsBAhQDFAAAAAgArR2HXHluM9foAAAArQEAABMAAAAAAAAAAAAAAIABAAAAAFtDb250ZW50X1R5cGVzXS54bWxQSwECFAMUAAAACACtHYdcm/036q0AAAApAQAACwAAAAAAAAAAAAAAgAEZAQAAX3JlbHMvLnJlbHNQSwECFAMUAAAACACtHYdc6fnBk3sAAACbAAAAHAAAAAAAAAAAAAAAgAHvAQAAd29yZC9fcmVscy9kb2N1bWVudC54bWwucmVsc1BLAQIUAxQAAAAIAK0dh1x4p6g08wAAAHgBAAARAAAAAAAAAAAAAACAAaQCAAB3b3JkL2RvY3VtZW50LnhtbFBLBQYAAAAABAAEAAMBAADGAwAAAAA=")
$xlsxBytes = [System.Convert]::FromBase64String("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")

function Write-File {
    param([string]$Path, [byte[]]$Bytes)
    $dir = Split-Path $Path -Parent
    if (-not (Test-Path $dir)) {
        [System.IO.Directory]::CreateDirectory($dir) | Out-Null
    }
    [System.IO.File]::WriteAllBytes($Path, $Bytes)
}

$clients = @(
        "Aldermoor Capital",
    "Brentfield Investments",
    "Caulfield Group",
    "Dunmore Asset Management",
    "Elsworth Partners",
    "Fenwick and Associates",
    "Graystone Wealth",
    "Harland Financial Services",
    "Ironbridge Fund Management",
    "Jennings Private Equity",
    "Kestrel Capital Partners",
    "Langton Wealth Management",
    "Meridian Asset Group",
    "Northgate Financial",
    "Oakfield Investment Trust",
    "Pemberton Capital",
    "Queensgate Advisors",
    "Riverstone Wealth",
    "Stanmore Investment Group",
    "Thornbury Asset Management",
    "Ullswater Capital",
    "Vantage Financial Partners",
    "Westmere Investments",
    "Xander Capital Group",
    "Yorkshire Asset Management",
    "Zephyr Wealth Advisors",
    "Ashford Capital",
    "Baxter Financial Group",
    "Clearwater Investments",
    "Dawnmere Capital",
    "Earlsfield Wealth",
    "Fairbourne Asset Management",
    "Greenvale Partners",
    "Hillcrest Financial Services",
    "Inverness Capital Group",
    "Juniper Wealth Management",
    "Kelburn Investment Partners",
    "Lakeshore Capital",
    "Montrose Asset Management",
    "Norbury Financial Group",
    "Oldfield Capital Partners",
    "Parkside Wealth Advisors",
    "Quantex Investment Group",
    "Redwood Financial Services",
    "Silverstone Capital",
    "Thornwick Wealth Management",
    "Upton Asset Group",
    "Veritas Capital Partners",
    "Westhaven Investments",
    "Ximena Financial Group",
    "Yarmouth Capital",
    "Zenith Wealth Management",
    "Ascot Financial Partners",
    "Beacon Capital Group",
    "Claremont Asset Management",
    "Dorset Wealth Advisors",
    "Elmwood Investments",
    "Farnsworth Capital",
    "Granville Asset Group",
    "Hadley Financial Partners",
    "Imperial Wealth Management",
    "Jericho Capital Group",
    "Kingsley Investments",
    "Linford Asset Management",
    "Marlowe Capital Partners",
    "Newbury Wealth Group",
    "Orchard Financial Services",
    "Pinnacle Asset Management",
    "Quinton Capital",
    "Radcliffe Wealth Advisors",
    "Sandstone Investment Group",
    "Telford Capital Partners",
    "Underwood Financial Group",
    "Vanthorpe Investments",
    "Whitmore Asset Management",
    "Excalibur Capital Group",
    "Yarborough Wealth Partners",
    "Zinc Financial Advisors",
    "Alston Capital Partners",
    "Blackmere Investments",
    "Crowfield Asset Management",
    "Dexford Financial Group",
    "Elmsworth Capital",
    "Falconer Wealth Management",
    "Goldsworth Investment Group",
    "Hartwell Capital Partners",
    "Inkstone Financial",
    "Jasper Asset Management",
    "Keynote Capital Group",
    "Lonsdale Wealth Advisors",
    "Montclair Investments",
    "Northfield Capital Partners",
    "Osprey Financial Group",
    "Pennington Asset Management",
    "Quorum Capital",
    "Rexford Wealth Management",
    "Saffron Investment Group",
    "Trentham Capital Partners",
    "Ulverston Financial",
    "Valdene Asset Management",
    "Westbrook Capital Group",
    "Xcel Wealth Partners"
)

$clientFolders = @(
    "01 - Onboarding", "02 - KYC and Compliance", "03 - Financial Statements",
    "04 - Tax", "05 - Correspondence", "06 - Agreements and Contracts"
)

$fileMap = @{
    "01 - Onboarding" = @(
        @{n="Client Onboarding Checklist";e="docx"},
        @{n="New Client Information Form";e="docx"},
        @{n="Fee Schedule Summary";e="xlsx"}
    )
    "02 - KYC and Compliance" = @(
        @{n="KYC Document Register";e="xlsx"},
        @{n="AML Risk Assessment";e="docx"},
        @{n="Identity Verification Notes";e="docx"}
    )
    "03 - Financial Statements" = @(
        @{n="FY2023 Balance Sheet";e="xlsx"},
        @{n="FY2023 Profit and Loss";e="xlsx"},
        @{n="FY2024 Q1 Management Accounts";e="xlsx"},
        @{n="FY2024 Q2 Management Accounts";e="xlsx"}
    )
    "04 - Tax" = @(
        @{n="FY2023 Tax Return Summary";e="docx"},
        @{n="Tax Provision Workings";e="xlsx"},
        @{n="GST Reconciliation";e="xlsx"}
    )
    "05 - Correspondence" = @(
        @{n="2024-03-15 Letter of Engagement";e="docx"},
        @{n="2024-06-01 Adviser Update";e="docx"},
        @{n="2024-09-10 Annual Review Summary";e="docx"}
    )
    "06 - Agreements and Contracts" = @(
        @{n="Service Agreement";e="docx"},
        @{n="Confidentiality Agreement";e="docx"},
        @{n="Investment Management Agreement";e="docx"}
    )
}

$internalFiles = @(
    @{f="_Internal\HR";                       n="Employee Register";           e="xlsx"},
    @{f="_Internal\HR";                       n="Leave Policy";                e="docx"},
    @{f="_Internal\HR";                       n="Staff Onboarding Checklist";  e="docx"},
    @{f="_Internal\Finance and Accounts";     n="FY2024 Budget";               e="xlsx"},
    @{f="_Internal\Finance and Accounts";     n="Monthly Expenses Tracker";    e="xlsx"},
    @{f="_Internal\Finance and Accounts";     n="Accounts Payable Register";   e="xlsx"},
    @{f="_Internal\Templates";                n="Letter Template";             e="docx"},
    @{f="_Internal\Templates";                n="Meeting Notes Template";      e="docx"},
    @{f="_Internal\Templates";                n="Report Template";             e="docx"},
    @{f="_Internal\Policies and Procedures";  n="Data Handling Policy";        e="docx"},
    @{f="_Internal\Policies and Procedures";  n="AML Compliance Procedure";    e="docx"},
    @{f="_Internal\Policies and Procedures";  n="Incident Response Procedure"; e="docx"}
)

Write-Host "Creating demo file structure ($($clients.Count) clients)..." -ForegroundColor Cyan

$i = 0
foreach ($client in $clients) {
    $i++
    Write-Host "[$i/$($clients.Count)] $client" -ForegroundColor Yellow
    foreach ($folder in $clientFolders) {
        foreach ($file in $fileMap[$folder]) {
            $path = Join-Path $root "Clients\$client\$folder\$($file.n).$($file.e)"
            $bytes = if ($file.e -eq "docx") { $docxBytes } else { $xlsxBytes }
            Write-File -Path $path -Bytes $bytes
        }
    }
}

Write-Host "Creating internal folders..." -ForegroundColor Cyan
foreach ($item in $internalFiles) {
    $path = Join-Path $root "$($item.f)\$($item.n).$($item.e)"
    $bytes = if ($item.e -eq "docx") { $docxBytes } else { $xlsxBytes }
    Write-File -Path $path -Bytes $bytes
}

Write-Host "`nDone! $root" -ForegroundColor Green
Write-Host "Clients: $($clients.Count) | Files per client: 16 | Total: $(($clients.Count * 16) + 12)" -ForegroundColor Green
```
