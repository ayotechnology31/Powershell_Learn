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

