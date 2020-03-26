function Create-ObjectHashTable($path)
{
    [System.Collections.Hashtable]$return = @{}
    $showHidden = 'n'
    #$showHidden = Read-Host "Show Hidden Folders? (y/n)"
    if ($showHidden -eq "y")
    {
        $allFiles = Get-ChildItem -force -Path $path  
    }
    else
    {
        $allFiles = Get-ChildItem -Path $path
    }
    $Length = $allFiles.Count

    for ($i=0; $i -le $Length -1 ; $i++)
    {
        $fileObject = New-Object -TypeName psobject
        $filePath = $allFiles[$i].FullName
        $name = $allFiles[$i].Name
        $mode = $allFiles[$i].Mode
        #$ErrorActionPreference = 'SilentlyContinue' 
        $num = ((Get-ChildItem -Force -Path $filePath -Recurse | Measure-Object -Property Length -Sum).Sum/1MB)
        $MB = "{0:N2}MB" -f $num
        #$GB = "{0:N2}G" -f ($num / 1024)
        $fileObject | Add-Member -MemberType NoteProperty -Name Name -Value $name
        $fileObject | Add-Member -MemberType NoteProperty -Name Path -Value $filePath
        $fileObject | Add-Member -MemberType NoteProperty -Name Size -Value $MB
        $fileObject | Add-Member -MemberType NoteProperty -Name Type -Value $mode
        $return.add($name, $fileObject)
    }
    $return
}


function Find-DiffrentFiles ($source, $destination)
{
    $source_hash = Create-ObjectHashTable -path $source
    $destination_hash = Create-ObjectHashTable -path $destination
    $return = New-Object 'System.Collections.Generic.List[System.Object]'

    foreach ($key in $source_hash.Keys)
    {    
        if($destination_hash.Keys -notcontains $key)
        {
            $return.Add($key)
        }
        else
        {
            if($destination_hash[$key].Size -ne $source_hash[$key].Size)
            {
                $return.add($key)
            }
        }
    }
    if ($return -eq $null)
    {
        $return = Compare-FileSize -source $source -destination $destination
    }
    $return
}

function Copy-Files ($source, $destination)
{
    $diffrentFiles = Find-DiffrentFiles -source $source -destination $destination
    foreach ($item in $diffrentFiles)
    {
        $source_hash = Create-ObjectHashTable -path $source
        $destination_hash = Create-ObjectHashTable -path $destination
        $SourceFullPath = $source_hash[$item].Path
        $destinationFullPath = $destination_hash[$item].Path

        $mode = $source_hash[$item] | Select-Object -ExpandProperty Type
        if ($mode -ne 'd-----' -or $destination_hash.Keys -notcontains $item)
        {
            Copy-Item -Path $SourceFullPath -Destination $destination -Force -Recurse
            Write-Host "File $item was Copped to $destination" -ForegroundColor Green
        }
        else
        {
            Copy-Files -source $SourceFullPath -destination $destinationFullPath
        }
    }
}



$source = Read-Host "Source Path"
$destination = Read-Host "Destination Path"
$source = $source -replace '\"' , ''
$destination = $destination -replace '\"' , ''

Copy-Files -source $source -destination $destination






