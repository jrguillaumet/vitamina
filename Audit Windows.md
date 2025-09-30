# Audit Windows

## Lista de Grupos locales / Miembros
Genera un fichero con la lista de los grupos locales y de dominio dentro de la maquina windows. Funciona en Workstation y Windows Server


```
$salida = ".\GruposLocalesYMiembros_$($env:COMPUTERNAME).csv"

try {
  Get-LocalGroup | ForEach-Object {
    $g = $_
    Get-LocalGroupMember -Group $g.Name -ErrorAction SilentlyContinue |
      Select-Object @{n='Servidor';e={$env:COMPUTERNAME}}, @{n='Grupo';e={$g.Name}}, Name, ObjectClass, PrincipalSource
  } | Sort-Object Grupo, Name | Export-Csv $salida -NoTypeInformation -Encoding UTF8
}
catch {
  ([ADSI]"WinNT://$env:COMPUTERNAME").Children |
    Where-Object { $_.SchemaClassName -eq 'group' } |
    ForEach-Object {
      $g = $_
      @($g.psbase.Invoke('Members')) | ForEach-Object {
        $name = $_.GetType().InvokeMember('Name','GetProperty',$null,$_,$null)
        [pscustomobject]@{
          Servidor = $env:COMPUTERNAME
          Grupo    = $g.Name
          Name     = $name
          Origen   = 'WinNT'
        }
      }
    } | Sort-Object Grupo, Name | Export-Csv $salida -NoTypeInformation -Encoding UTF8
}

Write-Host "CSV generado: $salida"

```
