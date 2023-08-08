# md-to-txt-powershell
Powershell script to turn md files into txt files

## 
1. Open terminal
2. Navigate to directory with md files
3. Copy and paste powershell script. The script puts the .txt files in `C:\users\<current users>` directory. It catches errors and skips files if the conversion takes 5 seconds.

```powershell
$markdownFiles = Get-ChildItem -Filter "*.md"
$totalFiles = $markdownFiles.Count
$currentFileNumber = 0

$runspace = [runspacefactory]::CreateRunspace()
$runspace.Open()

foreach ($file in $markdownFiles) {
    $currentFileNumber++
    $baseName = $file.BaseName

    # Display progress
    Write-Progress -Activity "Converting Markdown to Text" -PercentComplete (($currentFileNumber / $totalFiles) * 100) -Status "Processing" -CurrentOperation "$file"

    $psCmd = [powershell]::Create().AddScript({
        param($file, $baseName)
        & pandoc -s $file -o "$baseName.txt"
    }).AddArgument($file.FullName).AddArgument($baseName)

    $psCmd.Runspace = $runspace
    $handle = $psCmd.BeginInvoke()

    $timer = [Diagnostics.Stopwatch]::StartNew()
    while (-not $handle.IsCompleted -and $timer.Elapsed.TotalSeconds -lt 5) {
        Start-Sleep -Milliseconds 100
    }
    $timer.Stop()

    if (-not $handle.IsCompleted) {
        $psCmd.Stop()
        Write-Host "Conversion of $file timed out. Skipping..."
    } elseif ($psCmd.Streams.Error.Count -gt 0) {
        Write-Host "Error converting $file. Skipping..."
    }

    $psCmd.Dispose()
}

$runspace.Close()
$runspace.Dispose()

Write-Host "Conversion complete."
```

