Chapter 4. Running commands
That’s what you’ll be doing in this chapter: not scripting, not programming, but running commands and command-line utilities.

4.1. Not scripting, but running commandsWith PowerShell, you type a command,
add a few parameters to customize the command’s behavior, hit Return, and immediately see your results.
Give that file a .PS1 filename extension, and you suddenly have a “PowerShell script.” 
. All you need to do is master the ability to run commands within the shell, and you’re on your way

4.2 The anatomy of a command

4.3 The cmdlet naming convention
A cmdlet is a native PowerShell command-line utility. These exist only inside of
PowerShell and are written in a .NET Framework language like C#. The word
“cmdlet” is unique to PowerShell, so if you add it to your search keywords on
Google or Bing, the results you get back will be mainly PowerShell-related. The
word is pronounced “command-let.” 

 A function can be similar to a cmdlet, but rather than being written in a .NET
language, functions are written in PowerShell’s own scripting language.
 A workflow is a special kind of function that ties into PowerShell’s workflow execution system.
 An application is any kind of external executable, including command-line utilities like Ping, Ipconfig, and so forth.
 Command is the generic term that we use to refer to any or all of the preceding
terms.

 The rule is this: Names start with a standard verb, like Get or Set or New or Pause.
You can run Get-Verb to see a list of allowable verbs (you’ll see about 100, although
only about a dozen are common). After the verb is a dash, followed by a singular
noun, like Service or Process or EventLog. Developers get to make up their own
nouns, so there’s no “Get-Noun” cmdlet to display them all. 

 If you guessed “GetMailbox,” you got the first one right. If you guessed “Set-User,” you were close: it’s
Set-ADUser, and you’ll find the command on domain controllers in the ActiveDirectory module. The point is that by having this consistent naming convention with
a limited set of verbs, it becomes possible for you to guess at command names, and
you could then use Help or Get-Command, along with wildcards, to validate your guess.
It becomes easier for you to figure out the names of the commands you need, without
having to run to Google or Bing every time.

4.4 Aliases: nicknames for commands
 That’s where PowerShell aliases come in. An alias is nothing more than a nickname for a command. Tired of typing Get-Service? Try this:
PS C:\> get-alias -Definition "Get-Service"
Now you know that Gsv is an alias for Get-Service. 
 If you’re staring at an alias (folks on the internet tend to use them as if we’ve all
memorized all 150 built-in aliases) and can’t figure out what it is, ask help:
PS C:\> help gsv


Above and beyond
You can create your own aliases using New-Alias, export a list of aliases with
Export-Alias, or even import a list of previously created aliases using
Import-Alias. 
 When you create an alias, it only lasts as long as your current shell
session—once you close the window, it’s gone. That’s why you might want to export
them, so that you can more easily reimport them.

4.5 Taking shortcuts
4.5.1 Truncating parameter names
PowerShell doesn’t force you to type out entire parameter names. Instead of typing
-computerName, for example, you could go with -comp. The rule is that you have to type enough of the name for PowerShell to be able to disambiguate.  If there’s a
-computerName parameter, a -common parameter, and a -composite parameter, you’d
have to type at least -compu, -commo, and -compo, because that’s the minimum number
of letters necessary to uniquely identify each.

4.5.2 Parameter name aliases
Parameters can also have their own aliases, although they can be terribly difficult to
find, as they aren’t displayed in the help files or anyplace else convenient. For example, the Get-EventLog command has a -computerName parameter. To discover its
aliases, you’d run this command:
PS C:\> (get-command get-eventlog | select -ExpandProperty parameters).comp
utername.aliases

We’ve boldfaced the command and parameter names; replace these with whatever
command and parameter you’re curious about. In this case, the output reveals that
-Cn is an alias for -ComputerName, so you could run this:
PS C:\> Get-EventLog -LogName Security -Cn SERVER2 -Newest 10
Tab completion will show you the -Cn alias; if you typed Get-EventLog -C and started
pressing Tab, it’d show up. But the help for the command doesn’t display -Cn at all,
and Tab completion doesn’t indicate that -Cn and -ComputerName are the same thing

4.5.3 Positional parameters
When you’re looking at a command’s syntax in its help file, you can spot positional
parameters easily:
 For example, this is a bit difficult to read and interpret:
PS C:\> move file.txt users\donjones\
This version, which uses parameter names, is easier to follow:
PS C:\> move -Path c:\file.txt -Destination \users\donjones\
This version, which puts the parameters in a different order, is allowed when you use
the parameter names:
PS C:\> move -Destination \users\donjones\ -Path c:\file.txt

4.6 Cheating, a bit: Show-Command. 
If you’re having trouble getting a command’s syntax right, with
all the spaces, dashes, commas, quotes, and whatnot, Show-Command is your friend. 

4.7 Support for external commands
So far, all of the commands you’ve run in the shell (at least, the ones we’ve suggested
that you run) have been built-in cmdlets. Almost 400 of those cmdlets come built into
the latest version of the Windows client operating system, thousands into the server
operating system, and you can add more—products like Exchange Server, SharePoint
Server, and SQL Server all come with add-ins that each includes hundreds of additional cmdlets.

4.8 Dealing with errors
Sometimes, the error message might not seem helpful—and if it seems like you and
PowerShell are speaking different languages, you are. PowerShell obviously isn’t going
to change its language, so you’re probably the one in the wrong, and consulting the
help and spelling out the entire command, parameters and all, is often the quickest
way to solve the problem. And don’t forget to use Show-Command to try and figure out
the right syntax.

4.9 Common points of confusion
4.9.1 Typing cmdlet names
First up is the typing of cmdlet names. It’s always verb-noun, like Get-Content.
4.9.2 Typing parameters
Parameters are also consistently written. A parameter that takes no value, such as
-recurse, gets a dash before its name. You need to have spaces separating the
cmdlet name from its parameters, and the parameters from each other. The following are all correct:
 Dir -rec (the shortened parameter name is fine)
 New-PSDrive -name DEMO -psprovider FileSystem -root \\Server\Share

PowerShell isn’t normally picky about upper and lowercase, meaning that dir and DIR
are the same, as are -RECURSE and -recurse and -Recurse. But the shell sure is picky
about those spaces and dashes



4.10 Lab
NOTE For this lab, you’ll need a Windows 8 or Windows Server 2012 computer running PowerShell v3.
Using what you learned in this chapter, and in the previous chapter on using the help
system, complete the following tasks in Windows PowerShell:
1 Display a list of running processes. 
    get-process
2 Display the 100 most recent entries from the Application event log (don’t use
Get-WinEvent for this. We’ve shown you another command that will do this
task).
    get-eventlog -logname Applications -newest -100
3 Display a list of all commands that are of the “cmdlet” type (this is tricky—we’ve
shown you Get-Command, but you’re going to have to read the help to find out
how to narrow down the list, as we’ve asked).
    Get-Command -CommandType cmdlet
4 Display a list of all aliases.
    Get-Alias
5 Make a new alias, so you can run d to get a directory listing.
    New-Alias -Name np -Value notepad
6 Display a list of services that begin with the letter M. Again, read the help for the
necessary command—and don’t forget that the asterisk (*) is a near-universal
wildcard in PowerShell.
    Get-Service -Name m*
7 Display a list of all Windows Firewall rules. You’ll need to use Help or
Get-Command to discover the necessary cmdlet.
    Get-NetFirewallRule
8 Display a list only of inbound Windows Firewall rules. You can use the same
cmdlet as in the previous task, but you’ll need to read its help to discover the
necessary parameter and its allowable values.
    Get-NetFirewallRule -Direction Inbound


We hope these tasks seemed straightforward to you. If so—excellent. You were taking
advantage of your existing command-line skills to make PowerShell perform a few
practical tasks for you. If you’re new to the command-line world, these tasks are a
good introduction to what you’ll be doing in the rest of this book.
