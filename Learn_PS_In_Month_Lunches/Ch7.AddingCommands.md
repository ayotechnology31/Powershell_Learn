7.1 How one shell can do everything
We know you’re probably familiar with the graphical Microsoft Management Console (MMC), which is why we’ll use it as an example of how PowerShell works. The
two work similarly when it comes to extensibility, in part because the same Management Frameworks team within Microsoft develops both the MMC and PowerShell.
 When you open a new, blank MMC console, it’s largely useless. It can’t do anything, because the MMC has little built-in functionality. To make it useful, you go to
its File menu and select Add/Remove Snapins. In the MMC world, a snap-in is a
tool, such as Active Directory Users and Computers, DNS Management, DHCP
Administration, and so on. You can choose to add as many snap-ins to your MMC as
you like, and you can save the resulting console to make it easier to reopen that
same set of snap-ins in the future. 

7.2 About product-specific “management shells”
These product-specific management shells have been a huge source of confusion. We
want to clearly state that there is only one Windows PowerShell. There isn’t a separate
PowerShell for Exchange and Active Directory; it’s all a single shell. 

7.3 Extensions: finding and adding snap-ins
PowerShell v3 has two kinds of extensions: modules and snap-ins. 

A PSSnapin generally consists of one or more DLL files, accompanied by additional
XML files that contain configuration settings and help text. PSSnapins have to be
installed and registered in order for PowerShell to know they exist.
NOTE The PSSnapin concept is something Microsoft is moving away from,
and you’re likely to see fewer and fewer of them in the future. Internally,
Microsoft is focusing on delivering extensions as modules.
You can get a list of available snap-ins by running Get-PSSnapin -registered from
within PowerShell

 Once a snap-in is loaded, you’ll want to figure out what it added to the shell. A PSSnapin can add cmdlets, PSDrive providers, or both to the shell. To find out which
cmdlets you’ve added, use Get-Command (or its alias, Gcm):
PS C:\> gcm -pssnapin sqlservercmdletsnapin100


To see if the snap-in added any new PSDrive providers, run Get-PSProvider. You
can’t specify a snap-in with this cmdlet, so you’ll have to be familiar with the providers
that were already there, and scan through the list manually to spot anything new

Doesn’t look like anything new. We’re not surprised, because the snap-in we loaded
was named SqlServerCmdletSnapin100. If you recall, our list of available snap-ins also
included SqlServerProviderSnapin100, suggesting that the SQL Server team, for some
reason, packaged its cmdlets and PSDrive provider separately. Let’s try adding the second one:
PS C:\> add-pssnapin sqlserverprovidersnapin100
PS C:\> get-psprovider

7.4 Extensions: finding and adding modules
PowerShell v3 (and v2) supports a second type of extension called a module. Modules
are designed to be a little more self-contained, and somewhat easier to distribute, but
they work similarly to PSSnapins. But you do need to know a bit more about them in
order to find and use them.
 Modules don’t require advanced registration. Instead, PowerShell automatically
looks in a certain set of paths to find modules.

 The PSModulePath environment variable defines the paths where PowerShell expects modules to live:
PS C:\> get-content env:psmodulepath

As you can see in the previous example, there are two default locations: one in the
operating system folder, where system modules live, and one in the Documents folder,
where you can add any personal modules. You can also add a module from any other
location, provided you know its full path. 

NOTE PSModulePath isn’t something you can modify within PowerShell; it’s
set as part of your Windows environment. You can change it in the System
Control Panel, or you can set it via Group Policy.

run Get-Module | Remove-Module to remove any loaded modules.
PS C:\> help *network*

As you can see, PowerShell discovered several commands (of the “function” variety)
that have the word “network” in their name.
You could then ask for help on one of
these, even though you haven’t loaded the module:
PS C:\> help Get-SmbServerNetworkInterface

TIP You can also use Get-Module to retrieve a list of modules available on a
remote computer, and use Import-Module to load a remote module into your
current PowerShell session. You’ll learn how to do that in chapter 13 on
remote control.

PowerShell’s module auto-discovery enables the shell to complete command names
(using Tab in the console, or IntelliSense in the ISE), display help, and run commands, even for modules you haven’t explicitly loaded into memory. These features
make it worth the effort to keep PSModulePath complete and up to date.
 What if a module isn’t located in one of the paths referenced by PSModulePath?
You would need to run Import-Module and specify the complete path to the module,
such as C:\MyPrograms\Something\MyModule

If you have a Start menu shortcut for a product-specific shell—say, SharePoint
Server—and you don’t know where that product installed its PowerShell module,
open the properties for the Start menu shortcut. As we showed you earlier in this
chapter, the Target property of the shortcut will contain the Import-Module command used to load the module, and that will show you the module name and path.
 Modules can also add PSDrive providers. You’d use the same technique you used
for PSSnapins to identify any new providers: run Get-PSProvider. 

7.5 Command conflicts and removing extensions
Take a close look at the commands we added for both SQL Server and Active Directory. Notice anything special about the commands’ names?
 Most PowerShell extensions—Exchange Server being a notable exception—add a
short prefix to the noun portion of their command names. Get-ADUser, for example,
or Invoke-SqlCmd. These prefixes may seem awkward, but they’re designed to prevent
command conflicts. 

For example, suppose you loaded two modules that each contained a Get-User
cmdlet. With two commands having the same name and being loaded at the same time,
which one would PowerShell execute when you run Get-User? The last one loaded, as
it turns out. But the other commands having the same name aren’t inaccessible. To specifically run either command, you’d have to use a somewhat awkward naming convention that requires both the snap-in name and the command name. If one Get-User
came from a snap-in called MyCoolPowerShellSnapin, you’d have to run this:
MyCoolPowerShellSnapin\Get-User
That’s a lot of typing, and it’s why Microsoft suggests adding a product-specific prefix,
like AD or SQL, to the noun of each command. Adding prefixes helps prevent a conflict and helps make commands easier to identify and use. 
If you do wind up with a conflict, you can always choose to remove one of the conflicting extensions. You’d run Remove-PSSnapin or Remove-Module, along with the
snap-in or the module name, to unload an extension. 


7.6 Playing with a new module
Let’s put your newfound knowledge to use. We’ll assume that you’re using the newest
version of Windows, and we’d like you to follow along with the commands we present
in this section. More importantly, we want you to follow the process and the thinking
that we’ll explain, because this is how we teach ourselves to use new commands without rushing out and buying a new book for every single product and feature that we
run across. In the concluding lab for this chapter, we’ll have you repeat this same process on your own, to learn about an entirely new set of commands.
Our goal is to clear the DNS name resolution cache on our computer. We’ve no
idea if PowerShell can even do this, so we’ll start by asking the help system for a clue:
PS C:\> help *dns*

Ah-ha! As you can see, we have an entire DnsClient module on our computer. The
previous list shows the Clear-DnsClientCache command, but we’re curious about
what other commands are available. In order to find out, we’ll manually load the module and list its commands:

TRY IT NOW Go ahead and follow along as we run these commands. If you
don’t have a DnsClient module on your computer, then you’re using an older
version of Windows. Consider getting a newer version, or even a trial version
that you can run inside a virtual machine, so that you can follow along.
PS C:\> import-module -Name DnsClient
PS C:\> get-command -Module DnsClient

NOTE We could have asked for help on Clear-DnsClientCache, or even run
the command directly. PowerShell would have loaded the DnsClient module
for us in the background. But, because we’re exploring, this approach lets us
view the module’s complete list of commands.

This list of commands looks more or less the same as the earlier list. Fine; let’s see
what the Clear-DnsClientCache command looks like:
PS C:\> help Clear-DnsClientCache

NAME
 Clear-DnsClientCache
SYNTAX
 Clear-DnsClientCache [-CimSession <CimSession[]>] [-ThrottleLimit
 <int>] [-AsJob] [-WhatIf] [-Confirm] [<CommonParameters>]
Seems straightforward, and we don’t see any mandatory parameters. Let’s try running
the command:
PS C:\> Clear-DnsClientCache
VERBOSE: The specified name resolution records cached on this machine will
 be removed.
Subsequent name resolutions may return up-to-date information.
The -verbose switch is available for all commands, although not all commands do
anything with it. In this case, we get a message indicating what’s happening, which
tells us the command did run.

7.7 Profile scripts: preloading extensions
when the shell starts

Let’s say you’ve opened PowerShell, and you’ve loaded several favorite snap-ins and
modules. If you took that route, you’d be required to run one command for each
Licensed to <pedbro@gmail.com>
Profile scripts: preloading extensions when the shell starts 83
snap-in or module you want to load, which can take a few minutes of typing if you have
several of them. When you’re done using the shell, you close its window. The next
time you open a shell window, all of your snap-ins and modules are gone, and you
have to run all those commands again to load them back. Horrible. There must be a
better way.

We’ll show you three better ways. The first involves creating a console file. This only
memorizes PSSnapins that are loaded—it won’t work with any modules you may have
loaded. Start by loading in all of the snap-ins you want, and then run this command:
Export-Console c:\myshell.psc
Running the command creates a small XML file that lists the snap-ins you loaded into
the shell.
 Next, you’ll want to create a new PowerShell shortcut somewhere. The target of
that shortcut should be
%windir%\system32\WindowsPowerShell\v1.0\powershell.exe
➥-noexit -psconsolefile c:\myshell.psc

When you use that shortcut to open a new PowerShell window, your console will load,
and the shell will automatically add any snap-ins listed in that console file. Again, modules aren’t included. What do you do if you have a mix of snap-ins and modules, or if
you have some modules that you always want loaded?

TIP Keep in mind that PowerShell will auto-load modules that are in one of
the PSModulePath locations. You only need to worry about the following
steps if you want to preload modules that aren’t in one of the PSModulePath
locations.

The answer is to use a profile script. We’ve mentioned those before, and we’ll cover
them in more detail in chapter 25, but for now follow these steps to learn how to use
them:
1 In your Documents folder, create a new folder called WindowsPowerShell (no
spaces in the folder name).
2 In the newly created folder, use Notepad to create a file named profile.ps1.
When you save the file in Notepad, be sure to enclose the filename in quotation
marks (“profile.ps1”). Using quotes prevents Notepad from adding a .txt filename extension. If that .txt extension gets added, this trick won’t work.
3 In that newly created text file, type your Add-PSSnapin and Import-Module
commands, listing one command per line in order to load your preferred snapins and modules.
4 Back in PowerShell, you’ll need to enable script execution, which is disabled by
default. There are some security consequences to this that we’ll discuss in chapter 17 but for now we’ll assume you’re doing this in a standalone virtual
machine, or on a standalone test computer, and that security is less of an issue.
In the shell, run Set-ExecutionPolicy RemoteSigned. Note that the command
Licensed to <pedbro@gmail.com>
84 CHAPTER 7 Adding commands
will only work if you’ve run the shell as Administrator. It’s also possible for a
Group Policy object (GPO) to override this setting; you’ll get a warning message
if that’s the case.
5 Assuming you haven’t had any errors or warnings up to this point, close and
reopen the shell. It will automatically load profile.ps1, execute your commands,
and load your favorite snap-ins and modules for you.
TRY IT NOW Even if you don’t have a favorite snap-in or module yet, creating this simple profile will be good practice. If nothing else, put the command cd \ into the profile script, so that the shell always opens in the root
of your system drive. But please don’t do this on a computer that’s part of
your company’s production network, because we haven’t covered all of the
security implications yet.

7.8 Common points of confusion
PowerShell newcomers frequently do one thing incorrectly when they start working
with modules and snap-ins: they don’t read the help. Specifically, they don’t use the
-example or -full switches when asking for help.
 Frankly, looking at built-in examples is the best way to learn how to use a command. Yes, it can be a bit daunting to scroll through a list of hundreds of commands
(Exchange Server, for example, adds well over 400 new commands), but using Help
and Get-Command with wildcards should make it easier to narrow down the list to whatever noun you think you’re after. From there, read the help!
