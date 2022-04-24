# Powershell

Powershell cmdlets or commands have to be in verb-noun format. Ex: `get-command`

```
get-command -noun service // returns cmdlets with noun as service
```

To get help with a cmdlet

```
get-help <cmdlet>
get-help get-service
get-help new-service
Get-Help <cmdlet> -Detailed
Get-Help <cmdlet> -Full
Get-Help <cmdlet> -Examples
Get-Help <cmdlet> -ex
Get-Help <cmdlet> -Parameter *
Get-Help <cmdlet> -Parameter Name
    
help <cmdlet>
help new-service
```

In PS, all cmdlets return output as objects. Each object has methods and properties.

```
Get-Member \\ Gets the properties and methods of objects.

get-process \\ list all process
get-process -IncludeUserName
get-process <processname>
get-process -name WhatsApp | Get-Member
get-process -name WhatsApp | Select-Object *
```

```
PS C:\Users\adith\OneDrive\Desktop> get-process -IncludeUserName what*

Handles      WS(K)   CPU(s)     Id UserName               ProcessName
-------      -----   ------     -- --------               -----------
    490     215524    1,355   4700 TUF\ak                 WhatsApp
    393      16368       12  17712 TUF\ak                 WhatsApp
    760     221924      611  21760 TUF\ak                 WhatsApp
    311      16800        2  21988 TUF\ak                 WhatsApp
    287       4624        1  23784 TUF\ak                 WhatsApp
   1006      70884      199  27604 TUF\ak                 WhatsApp
    189       2688        0  28252 TUF\ak                 WhatsApp
```

### Variables

variables can be defined with **$** sign

```
PS C:\Users\adith\OneDrive\Desktop> $whatsapp = Get-Process Whatsapp
PS C:\Users\adith\OneDrive\Desktop> $whatsapp

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    490      41   307920     265112      1,356   4700  11 WhatsApp
    304      18    17668      16012         12  17712  11 WhatsApp
    758      37   389332     228692        611  21760  11 WhatsApp
    311      22    55420      16760          2  21988  11 WhatsApp
    284      18    10872       4600          1  23784  11 WhatsApp
   1004      45   337392      69600        199  27604  11 WhatsApp
    189      12     9356       2688          0  28252  11 WhatsApp
    
PS C:\Users\adith\OneDrive\Desktop> $whatsapp.Count
7
```

Storing objects inside variables and accessing the method of those stored objects

```
PS C:\Users\adith\OneDrive\Desktop> $myyphone = get-process yourphone
PS C:\Users\adith\OneDrive\Desktop> $myphone

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
   1094      72    62524      79220          2  15912  11 YourPhone

PS C:\Users\adith\OneDrive\Desktop> $myphone.Kill()
PS C:\Users\adith\OneDrive\Desktop>
```
