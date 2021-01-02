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