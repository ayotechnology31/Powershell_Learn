Working with providers
5.1 What are providers?
A PowerShell provider, or PSProvider, is an adapter. It’s designed to take some kind
of data storage and make it look like a disk drive. 

Notice that each provider has different capabilities. This is important, because it
affects the ways in which you can use each provider. These are some of the common
capabilities you’ll see:
 ShouldProcess—Means the provider supports the use of the -WhatIf and
-Confirm parameters, enabling you to “test” certain actions before committing
to them.
 Filter—Means the provider supports the -Filter parameter on the cmdlets
that manipulate providers’ content.
 Credentials—Means the provider permits you to specify alternate credentials
when connecting to data stores. There’s a -credential parameter for this.
 Transactions—Means the provider supports the use of transactions, which
allows you to use the provider to make several changes, and then either roll
back or commit those changes as a single unit.
You use a provider to create a PSDrive. A PSDrive uses a single provider to connect to
some actual data storage. You’re essentially creating a drive mapping, much like you
might have in Windows Explorer, but a PSDrive, thanks to the providers, is able to
connect to much more than disks. Run the following command to see a list of currently connected drives:
PS C:\> Get-PSDrive

5.2 How the filesystem is organized
The Windows filesystem is organized around three main types of objects: drives, folders, and files.
Drives, the top-level objects, contain both folders and files. 
Folders are also a kind of container, capable of containing both files and other folders. 
Files aren’t a type of container; they’re more of an endpoint object. 
You’re probably most familiar with viewing the filesystem through Windows
Explorer, as shown in figure 5.1, where the hierarchy of drives, folders, and files is
visually obvious.

PowerShell doesn’t use the terms
“file” and “folder.” Instead, it refers to these objects by the more generic term item.
Both a file and a folder are considered items, although they’re obviously different
types of items. That’s why the cmdlet names we showed you previously all use the word
“item” in their noun.

 Items can, and often do, have properties.

 Verbs like Clear, Copy, Get, Move, New, Remove, Rename, and Set can all apply
to items (like files and folders) as well as to item properties (such as the date
the item was last written, or whether it’s read-only).
 The Item noun refers to individual objects, like files and folders.
 The ItemProperty noun refers to attributes of an item, such as read-only, creation time, length, and so on.
 The ChildItem noun refers to the items (like files and subfolders) contained
within an item (like a folder).

 Some PSProviders don’t support item properties. For example, the Environment
PSProvider is what’s used to make the ENV: drive available in PowerShell.

The fact that not every PSProvider is the same is perhaps what makes providers so confusing for PowerShell newcomers. You have to think about what each provider is giving you access to, and understand that even when the cmdlet knows how to do
something, that doesn’t mean the particular provider you’re working with will support
that operation.

5.3 How the filesystem is like other data stores
The filesystem is a model for other forms of storage. For example, figure 5.2 shows the
Windows Registry Editor.
 The registry is laid out much like the filesystem with folders (registry keys), files
(registry values), and so on. It’s this broad similarity that makes the filesystem the perfect model, which is why PowerShell connects to data stores as drives, exposing items
and item properties. 
 When you dig into the
details, the various different forms of storage are quite different. That’s why the various “item” cmdlets support such a broad range of functionality, and why not every bit
of functionality will work with every possible form of storage

5.4 Navigating the filesystem
Another cmdlet you’ll need to know when working with providers is Set-Location.
It’s what you use to change the shell’s current location to a different container-type
item, such as a folder:
PS C:\> Set-Location -Path C:\Windows
PS C:\Windows>

e New-Item cmdlet is generic—it doesn’t know you want to create a
folder. It can create folders, files, registry keys, and much more, but you have to tell it
what type of item you want to create:
PS C:\users\donjones\Documents> new-item testFolder
Type: directory
 Directory: C:\
PS C:\users\donjones\Documents> new-item testFolder
Type: directory
 Directory: C:\users\donjones\Documents
Mode LastWriteTime Length Name
---- ------------- ------ ----
d---- 3/29/2012 10:43 AM testFolder

PowerShell does include a Mkdir command, which most people think is an alias to
New-Item. But using Mkdir doesn’t require you to enter a type:
PS C:\users\donjones\Documents> mkdir test2
 Directory: C:\users\donjones\Documents

What gives? It turns out that Mkdir is a function, not an alias. Internally, it still uses
New-Item, but the function adds the -Type Directory parameter for you, making
Mkdir behave more like its Cmd.exe predecessor

5.5 Using wildcards and literal paths
Most of the “item” cmdlets include a -Path property, and by default that property
accepts wildcards. Looking at the full help for Get-ChildItem, for example, reveals
the following:

-Path <String[]>
 Specifies a path to one or more locations. Wildcards are
 permitted. The default location is the current directory (.).
 Required? false
 Position? 1
 Default value Current directory
 Accept pipeline input? true (ByValue, ByPropertyName)
 Accept wildcard characters? True

The * wildcard stands in for zero or more characters, whereas the ? wildcard stands in
for any single character. You’ve doubtless used this time and time again, probably with
the Dir alias of Get-ChildItem:
Licensed to <pedbro@gmail.com>
56 CHAPTER 5 Working with providers
PS C:\Windows> dir *.exe

 PowerShell’s solution is to provide an alternate -LiteralPath parameter.
When you want * and ? taken literally, you use -LiteralPath instead of the -Path
parameter. Note that -LiteralPath isn’t positional; if you plan to use it, you have to
type -LiteralPath. 


5.6 Working with other providers
One of the best ways to get a feel for these other providers, and how the various “item”
cmdlets work, is to play with a PSDrive that isn’t the filesystem. Of the providers built
into PowerShell, the registry is probably the best example to work with (in part
because it’s available on every system). Our goal is to turn off the “Aero Peek” feature
in Windows.
 Start by changing to the HKEY_CURRENT_USER portion of the registry, exposed by
the HKCU: drive:
PS C:\> set-location -Path hkcu:
Next, navigate to the right portion of the registry:
PS HKCU:\> set-location -Path software
PS HKCU:\software> get-childitem
 Hive: HKEY_CURRENT_USER\software
Name Property
---- --------
AppDataLow
clients
Microsoft
Mine (default) : {}
Parallels
Policies
PS HKCU:\software> set-location microsoft
PS HKCU:\software\microsoft> Get-ChildItem
 Hive: HKEY_CURRENT_USER\software\microsoft
Name Property
---- --------
.NETFramework
Active Setup
Advanced INF Setup
Assistance
AuthCookies
Command Processor PathCompletionChar : 9
 EnableExtensions : 1
 CompletionChar : 9
 DefaultColor : 0
CTF
EventSystem
Fax
Feeds SyncTask : User_Feed_Synchronization-{28B6
 C75-A5AB-40F7-8BCF-DC87CA308D51
 }
FTP Use PASV : yes
IdentityCRL UpdateDone : 1
Immersive Browser
Internet Connection Wizard Completed : 1
Internet Explorer
Keyboard

MediaPlayer
Microsoft Management Console
MSF
PeerNet
RAS AutoDial
Remote Assistance
Speech
SQMClient UserId :
 {73C1117E-B151-4C82-BA8D-BFF6134D1E10}
SystemCertificates
TabletTip
WAB
wfs
Windows
Windows Mail Setup DelayStartTime : {186, 248, 138, 82...}
 DelayInitialized : 2
Windows Media
Windows NT
Windows Script
Windows Script Host
Windows Search
Windows Sidebar
Wisp
You’re almost finished. You’ll notice that we’re sticking with full cmdlet names rather
than using aliases to emphasize the cmdlets themselves:
PS HKCU:\software\microsoft> Set-Location .\Windows
PS HKCU:\software\microsoft\Windows> Get-ChildItem
 Hive: HKEY_CURRENT_USER\software\microsoft\Windows
Name Property
---- --------
CurrentVersion
DWM Composition : 1
 EnableAeroPeek : 1
 AlwaysHibernateThumbnails : 0
 ColorizationColor :
 3226847725
 ColorizationColorBalance : 72
 ColorizationAfterglow :
 3226847725
 ColorizationAfterglowBalance : 0
 ColorizationBlurBalance : 28
 ColorizationGlassReflectionIntensity : 50
 ColorizationOpaqueBlend : 0
 ColorizationGlassAttribute : 0
Roaming
Shell
TabletPC
Windows Error Reporting Disabled : 0
 MaxQueueCount : 50
 DisableQueue : 0
 LoggingDisabled : 0
 DontSendAdditionalData : 0
 ForceQueue : 0
  DontShowUI : 0
  ConfigureArchive : 1
  MaxArchiveCount : 500
  DisableArchive : 0
  LastQueuePesterTime : 129773462733828600
 Note the EnableAeroPeek registry value. Let’s change it to 0:
 PS HKCU:\software\microsoft\Windows> Set-ItemProperty -Path dwm -PSPropert
  EnableAeroPeek -Value 0
 Let’s check it again to make sure the change “took”:
 PS HKCU:\software\microsoft\Windows> Get-ChildItem
  Hive: HKEY_CURRENT_USER\software\microsoft\Windows
 Name Property
 ---- --------
 CurrentVersion
 DWM Composition : 1
  EnableAeroPeek : 0
  AlwaysHibernateThumbnails : 0
  ColorizationColor :
  3226847725
  ColorizationColorBalance : 72
  ColorizationAfterglow :
  3226847725
  ColorizationAfterglowBalance : 0
  ColorizationBlurBalance : 28
  ColorizationGlassReflectionIntensity : 50
  ColorizationOpaqueBlend : 0
  ColorizationGlassAttribute : 0
 Roaming
 Shell
 TabletPC
 Windows Error Reporting Disabled : 0
  MaxQueueCount : 50
  DisableQueue : 0
  LoggingDisabled : 0
  DontSendAdditionalData : 0
  ForceQueue : 0
  DontShowUI : 0
  ConfigureArchive : 1
  MaxArchiveCount : 500
  DisableArchive : 0
  LastQueuePesterTime : 129773462733828600
 Mission accomplished! Using these same techniques, you should be able to work with
 any provider that comes your way.
 
 

5.7 Lab
NOTE For this lab, you’ll need any computer running PowerShell v3.
Complete the following tasks:
1 In the registry, go to HKEY_CURRENT_USER\software\microsoft\Windows\
currentversion\explorer. Locate the Advanced key, and set its DontPrettyPath
property to 1.
    cd HKCU:\software\microsoft\Windows\currentversion\explorer
    cd advanced
    Set-ItemProperty -Path . Name DontPrettyPath -Value 1
    
    
2 Create a new directory called C:\Labs
    mkdir c:\labs
    or the New-Item cmdlet:
    new-item =path C:\Labs -ItemType Directory
    
3 Create a zero-length file named C:\Test.txt (use New-Item).
New-Item -path c:\labs -Name test.txt -ItemType file
    
4 Is it possible to use Set-Item to change the contents of C:\Test.txt to TESTING?
Or do you get an error? If you get an error, why?
    The filesysterm provider doesn't support this action. 
    
5 Using Environment provider, display the value of the system environment variable %TEMP%
    Either of these commands works: Get-item env:temp OR Dir env:temp
    
6 What are the differences between the -Filter, -Include, and -Exclude parameters of Get-ChildItem?
    Include and Exclude must be used with -Recurse or if querying a container. Filter uses the PS Provider's filter capability, which not all providers support. For example, you could use DIR -filter in the file system but not in the Register -although you could use DIR -include inthe Registry to achieve almost the same type of filtering result

