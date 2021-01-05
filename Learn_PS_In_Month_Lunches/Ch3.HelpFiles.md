3.1. The help system: how you discover commands
RTFM: Read the friendly manual
Take time to read the help files
-Help system helps you find commands
-Help sytems shows you how to run the commands so ou dont get errors
-Link commands together to perform a complex task

 COMMAND VS. CMDLET
PowerShell contains many types of executable commands. 
Some are called cmdlets, some are called functions, others are known as workflows, and so on. 
Collectively, they’re all commands, and the help system works with all of them. 
A cmdlet is something unique to PowerShell, and many of the commands you run will be cmdlets. 
But we’ll try to consistently use command whenever we’re talking about the more general class of executable utility.

3.2. Updatable help
Help > Update Windows PS 
Update every month or so

Do you have computers that aren’t connected to the internet? 
No problem: Go to one that’s connected, and use Save-Help to get a local copy of the help. 
Put it on a file server or somewhere that’s accessible to the rest of your network. 
Then run Update-Help with its -Source parameter, pointing it to the downloaded copy of the help. 
That’ll let any computer on your network grab the updated help from that central spot, rather than from the internet.

If you have PC's that arent connected to the internet,GO to one 
Help is open sourced: Microsoft’s PowerShell help files are open source materials that are available at http://github.com/powershell. 

3.3. Asking for help
cmdlet provides Get-Help keyword byt some examples show "help" keyword instead or even the "man" keyword from "manual".
"Man" and "Help" arent native cmdlets at all; they are functions which are wrappers around the core "Get-Help"

HELP ON MACOS/LINUX
It uses "man". Takes over the screen to display help and returning to your normal scren when you're finished.

Help works much like the base Get-Help, but it pipes the help output to More, allowing you to have a nice paged view instead of seeing all the help fly by at once. Running Help Get-Content and Get-Help Get-Content produces the same results, but the former has a page-at-a-time display. You could run Get-Help Get-Content | More to produce that paged display, but that requires a lot more typing. We typically use only Help, but we want you to understand that some trickery is going on under the hood.

NOTE
The More command won’t work in the ISE. Even if you use Help or Man, the help content displays all at once rather than a page at a time.

The help system has two main goals: 
to help you find commands to perform specific tasks, 
and to help you learn how to use those commands after you’ve found them.

3.4. Using help to find commands
Do something with event log:
Help *log*
Help *event*
Result is a list like this:
Name                              Category  Module                    Synopsis                                    
----                              --------  ------                    --------                                    
Clear-EventLog                    Cmdlet    Microsoft.PowerShell.M... Clears all entries from specified event l...

Whenever possible, try to search using the broadest term possible—*event* or *log* as opposed to *eventlog*—because you’ll get the most results possible.

Don’t forget about Tab completion! As a reminder, it lets you type a portion of a command name and press Tab, to have the shell complete what you’ve typed with the closest match. You can continue pressing Tab to cycle through alternative matches.

Using *wildcard which stands in for zero or more characters -with "Help".
Run Help Get-EventL* and you should see the help file for Get-EventLog, rather than a list of matching help topics.

Get-EventLog. This file, called the summary help, is meant to be a short description of the command and a reminder of the syntax. 

ABOVE AND BEYOND
We mentioned that the Help command doesn’t search for cmdlets; it searches for help topics. Because every cmdlet has a help file, we could say that this search retrieves the same results. But you can also directly search for cmdlets by using the Get-Command cmdlet (or its alias, Gcm).

3.5. Interpreting the help
3.5.1. Parameter sets and common parameters

Get-EventLog

SYNTAX
    Get-EventLog [-AsString] [-ComputerName <string[]>] [-List] [<Com

    monParameters>]
    Get-EventLog [-LogName] <string> [[-InstanceId] <Int64[]>] [-Afte
    r <DateTime>] [-AsBaseObject] [-Before <DateTime>] [-ComputerName
     <string[]>] [-EntryType <string[]>] [-Index <Int32[]>] [-Message
     <string>] [-Newest <int>] [-Source <string[]>] [-UserName <strin
    g[]>] [<CommonParameters>]

 command supports two parameter sets; you can use the command in two distinct ways. 

You’ll notice that every parameter set for every PowerShell cmdlet ends with [<CommonParameters>]. 

3.5.2. Optional and mandatory parameters
You don’t need every single parameter in order to make a cmdlet run. PowerShell’s help lists optional parameters in square brackets. For example, [-ComputerName <string[]>] indicates that the entire -ComputerName parameter is optional. 
 You don’t have to use it at all; the cmdlet will probably default to the local computer if you don’t specify an alternative name using this parameter. That’s also why [<-Common-Parameters>] is in square brackets: You can run the command without using any of the common parameters.
Follow along in this example by running Get-EventLog without any parameters.
PowerShell should have prompted you for the mandatory LogName parameter. If you type something like System or Application and hit Enter, the command will run correctly. You could also press Ctrl-C to abort the command.

3.5.3. Positional parameters
Those commonly used parameters are often positional: You can provide a value without typing the parameter’s name, provided you put that value in the correct position.

You can identify positional parameters in two ways: via the syntax summary or the full help.

Finding positional parameters in the syntax summary
You’ll find the first way in the syntax summary: the parameter name—only the name—will be surrounded by square brackets.

Finding positional parameters in the full help
We said that you can locate positional parameters in two ways. The second requires that you open the help file by using the -full parameter of the Help command.

3.5.5. Finding command examples
Help Get-EventLog -example

3.6. Accessing “about” topics
Help *common*
About_common_parameters

3.7. Accessing online help
Help Get-EventLog -online

3.8 Lab
NOTE For this lab, you’ll need any computer running PowerShell v3.
We hope this chapter has conveyed the importance of mastering the help system in
PowerShell. Now it’s time to hone your skills by completing the following tasks. Keep
in mind that sample answers can be found on MoreLunches.com. Look for italicized
words in these tasks, and use them as clues to complete that task.
1 First, run Update-Help and ensure it completes without errors. That will get a
copy of the help on your local computer. You’ll need an internet connection,
and the shell needs to run under elevated privileges (which means it must say
“Administrator” in the shell window’s title bar).
2 Can you find any cmdlets capable of converting other cmdlets’ output into
HTML?
3 Are there any cmdlets that can redirect output into a file, or to a printer?
4 How many cmdlets are available for working with processes? (Hint: remember
that cmdlets all use a singular noun.)
5 What cmdlet might you use to write to an event log?
6 You’ve learned that aliases are nicknames for cmdlets; what cmdlets are available to create, modify, export, or import aliases?
7 Is there a way to keep a transcript of everything you type in the shell, and save
that transcript to a text file?
8 It can take a long time to retrieve all of the entries from the Security event log.
How can you get only the 100 most recent entries?
9 Is there a way to retrieve a list of the services that are installed on a remote
computer?
10 Is there a way to see what processes are running on a remote computer?
11 Examine the help file for the Out-File cmdlet. The files created by this cmdlet
default to a width of how many characters? Is there a parameter that would
enable you to change that width?
12 By default, Out-File will overwrite any existing file that has the same filename
as what you specify. Is there a parameter that would prevent the cmdlet from
overwriting an existing file?
13 How could you see a list of all aliases defined in PowerShell?
14 Using both an alias and abbreviated parameter names, what is the shortest command line you could type to retrieve a list of running processes from a computer named Server1?
15 How many cmdlets are available that can deal with generic objects? (Hint:
remember to use a singular noun like “object” rather than a plural one like
“objects”.)
16 This chapter briefly

3.9 Lab answers
1. Update-help
Or if you run it more than once in a single day: Update-Help -force
2. help html Or you could try with Get-Command: get-command -noun html
3. get-command -noun file, printer
4. get-command -noun process or Help *Process
5. Get-command -noun process or Help *Process
5. get-command -verb write -noun eventlog
Or if you aren't sure about the noun, use wildcard: help *log
6. help *alias or get-command -noun alias
7. help transcript
8. help Get-Eventlog -parameter Newst
9. help Get-Service -parameter computername
10. help Get-Process -parameter computername
11. Help Out-File -full or Help Out-File -parameter Width, should show you 80 characters as the default for the PowerShell console. You would use this parameter to change it as well.
12. If you run Help Out-File -full and loook at parameters, you should see -NoCobber.
13. Get-Alias
14. ps -c server1
15. get-command -noun object
16. help about_arrays Or you could use wildcards: help *array
