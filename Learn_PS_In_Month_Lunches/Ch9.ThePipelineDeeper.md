Chapter 9: The pipeline, deeper

At this point, you’ve learned to be pretty effective with PowerShell’s pipeline. Running commands (like Get-Process | Sort VM -desc | ConvertTo-HTML | Out-File
procs.html) is powerful, accomplishing in one line what used to take several lines
of script. But you can do even better. In this chapter, we’ll dig deeper into the pipeline and uncover some of its most powerful capabilities.

9.1 The pipeline: enabling power with less typing
One of the reasons we like PowerShell so much is that it enables us to be more
effective administrators without having to write complex scripts, like we used to
have to do in VBScript. But the key to powerful one-line commands lies in the way
the PowerShell pipeline works.

Let us be clear: You could skip this chapter and still be effective with PowerShell,
but you would in most cases have to resort to VBScript-style scripts and programs.
Although PowerShell’s pipeline capabilities can be complicated, they’re probably
easier to learn than more-complicated programming skills. By learning to manipulate the pipeline, you can be much more effective without needing to write scripts.
 The whole idea here is to get the shell to do more of your work for you, with as
little typing as possible. We think you’ll be surprised at how well the shell can do that!


9.2 How PowerShell passes data down the pipeline
Whenever you string two commands together, PowerShell has to figure out how to
get the output of the first command to the input of the second command. In the
upcoming examples, we’re going to refer to the first command as Command A.

Figure 9.1 Creating a text file
containing computer names

That’s the command that produces something. The second command will be Command B, which needs to accept Command A’s output and then do its own thing.
PS C:\> CommandA | CommandB

For example, suppose you have a text file that contains one computer name on each
line, as shown in figure 9.1.
 You might want to use those computer names as the input to some command, telling that command which computers you want it to run against. Consider this example:
PS C:\> Get-Content .\computers.txt | Get-Service
When Get-Content runs, it places the computer names into the pipeline. PowerShell
then has to decide how to get those to the Get-Service command. The trick with
PowerShell is that commands can only accept input on a parameter. That means
PowerShell has to figure out which parameter of Get-Service will accept the output
of Get-Content. This figuring-out process is called pipeline parameter binding, and it’s
what we’ll be covering in this chapter. PowerShell has two methods it can use to get
the output of Get-Content onto a parameter of Get-Service. The first method the
shell will try is called ByValue; if that doesn’t work, it’ll try ByPropertyName.


9.3 Plan A: pipeline input ByValue
With this pipeline parameter binding method, PowerShell looks at the type of object
produced by Command A and tries to see if any parameter of Command B can accept
that type of object from the pipeline. You can determine this for yourself: first, pipe
the output of Command A to Get-Member, to see what type of object Command A is
producing. Then, examine the full help of Command B (for example, Help
Get-Service -full) to see if any parameter accepts that type of data from the pipeline ByValue. Figure 9.2 shows what you might discover.
 What you’ll find is that Get-Content produces objects of the type System.String
(or String for short). You’ll also find that Get-Service does have a parameter that
accepts String from the pipeline ByValue. The problem is that it’s the -Name parameter, which according to the help “specifies the service names of services to be
retrieved.” That isn’t what we wanted—our text file, and therefore our String objects,
are computer names, not service names. If we ran the following,
PS C:\> Get-Content .\computers.txt | Get-Service
we’d be attempting to retrieve services named SERVER2, WIN8, and so forth, which is
probably not going to work.
 PowerShell only permits one parameter to accept a given type of object from the
pipeline ByValue. This means that because the -Name parameter accepts String from
the pipeline ByValue, no other parameter can do so. That dashes our hopes for trying
to pipe computer names from our text file to Get-Service.
 In this case, pipeline input is working, but it isn’t achieving the results we’d hoped
for. Let’s consider a different example, where we do get the results we want. Here’s
the command line:
PS C:\> get-process -name note* | Stop-Process

Figure 9.3 Binding the output of Get-Process to a parameter of Stop-Process

Let’s pipe the output of Command A to Get-Member and examine the full help for
Command B. Figure 9.3 shows what you’ll find.
Get-Process produces objects of the type System.Diagnostics.Process (note
that we limited the command to processes whose names start with note*; we made sure
a copy of Notepad was running so that the command would produce some output).
Stop-Process can accept those Process objects from the pipeline ByValue; it does so
on its -InputObject parameter. According to the help, that parameter “stops the processes represented by the specified process objects.” In other words, Command A will
get one or more Process objects, and Command B will stop (or kill) them.
 This is a good example of pipeline parameter binding in action, and it also illustrates an important point in PowerShell: For the most part, commands sharing the
same noun (as Get-Process and Stop-Process do) can usually pipe to each other
ByValue.
 Let’s cover one more example:
PS C:\> get-service -name s* | stop-process
Figure 9.4 Examining the output of Get-Service and the input parameters of Stop-Process

On the face of it, this might not seem to make any sense. But let’s see this through by
piping Command A’s output to Get-Member, and re-examining the help for Command
B. Figure 9.4 shows what you should find.
Get-Service produces objects of the type ServiceController (technically, System
.ServiceProcess.ServiceController, but you can usually take the last bit of the
TypeName as a shortcut). Unfortunately, there isn’t a single parameter of Stop-Process
that can accept a ServiceController object. That means the ByValue approach has
failed, and PowerShell will try its backup plan: ByPropertyName.


9.4 Plan B: pipeline input ByPropertyName
With this approach, you’re still looking to attach the output of Command A to parameters of Command B. But ByPropertyName is slightly different than ByValue. With this
backup method, it’s possible for multiple parameters of Command B to become
involved. Once again, pipe the output of Command A to Get-Member, and then look at
the syntax for Command B. Figure 9.5 shows what you should find: The output of Command A has one property whose name corresponds to a parameter on Command B.
    A lot of folks will overthink what’s happening here, so let’s be clear on how simple
the shell is being: it’s literally looking for property names that match parameter
names. That’s it. Because the property “Name” is spelled the same as the parameter
“-Name,” the shell will try to connect the two. 

Figure 9.5 Mapping properties to parameters

But it can’t do so right away: first it needs to see if the -Name parameter will accept
input from the pipeline ByPropertyName. A glance at the full help, shown in
figure 9.6, is required to make this determination.

In this case, -Name does accept pipeline input ByPropertyName, so this connection
will work. Now, here’s the trick: unlike ByValue, where only one parameter would be
involved, ByPropertyName will connect every matching property and parameter (provided each parameter has been designed to accept pipeline input ByPropertyName).
Figure 9.6 Checking to see if Stop-Process’s -Name parameter accepts pipeline input ByPropertyName

Figure 9.7 Attempting to pipe Get-Service to Stop-Process

  A bunch of error messages. The problem is that services’ names are usually things
like ShellHWDetection and SessionEnv, whereas the services’ executables might be
things like svchost.exe. Stop-Process only deals with those executable names. But
even though the Name property connects to the -Name parameter via the pipeline, the
values inside the Name property don’t make sense to the -Name parameter, which leads
to the errors.
  Let’s look at a more successful example. Create a simple comma-separated values
(CSV) file in Notepad, using the example in figure 9.8.
Save the file as Aliases.csv. Now, back in the shell, try importing it, as shown in
figure 9.9. You should also pipe the output of Import-CSV to Get-Member, so that you
can examine the output’s members.

Figure 9.8 Create this CSV file
in Windows Notepad.


Figure 9.9 Importing the CSV file and checking its members

Figure 9.10 Matching properties to parameter names

You can clearly see that the columns from the CSV file become properties, and each
data row in the CSV file becomes an object. Now, examine the help for New-Alias, as
shown in figure 9.10.
 Both of the properties—Name and Value—correspond to parameter names of NewAlias. Obviously, this was done on purpose—when you create the CSV file, you can
name those columns anything you want. Now, check to see if -Name and -Value accept
pipeline input ByPropertyName, as shown in figure 9.11.
 Both parameters do, meaning this trick will work. Try running the command:
PS C:\> import-csv .\aliases.csv | new-alias
The result will be three new aliases, named d, sel, and go, which point to the commands Get-ChildItem, Select-Object, and Invoke-Command, respectively. This is a
powerful technique for passing data from one command to another, and for accomplishing complex tasks in a minimum number of commands.

Figure 9.11 Looking for parameters that accept pipeline input ByPropertyName

9.5 When things don’t line up: custom properties
The CSV example was cool, but it’s pretty easy to make property and parameter names
line up when you’re creating the input from scratch. Things get tougher when you’re
forced to deal with objects that are created for you, or data that’s being produced by
someone else.
 For this example, we’re going to introduce a new command that you might not
have access to: New-ADUser. It’s part of the ActiveDirectory module, which you’ll find
on any Windows Server 2008 R2 (or later) domain controller. You can also get that
module on a client computer by installing Microsoft’s Remote Server Administration
Tools (RSAT). But for now, don’t worry about running the command; follow along
with the example.
New-ADUser has a number of parameters, each designed to accept information
about a new Active Directory user. Here are some examples:
 -Name (this is mandatory)
 -samAccountName (technically not mandatory, but you have to provide it to
make the account usable)
-Department
 -City
 -Title
We could cover the others, but let’s work with these. All of them accept pipeline input
ByPropertyName.

Figure 9.12 Working with
the CSV file provided by
Human Resources

As you can see in figure 9.12, the shell can import the CSV file fine, resulting in three
objects with four properties apiece. The problem is that the dept property won’t line
up with the -Department parameter of New-ADUser, the login property is meaningless, and you don’t have samAccountName or Name properties—both of which are
required if you want to be able to run this command to create new users:
PS C:\> import-csv .\newusers.csv | new-aduser
How can you fix this? Obviously, you could open the CSV file and fix it, but that’s a lot
of manual work over time, and the whole point of PowerShell is to reduce manual
labor. Why not set up the shell to fix it instead? Look at the following example:
PS C:\> import-csv .\newusers.csv |
>> select-object -property *,
>> @{name='samAccountName';expression={$_.login}},
>> @{label='Name';expression={$_.login}},
>> @{n='Department';e={$_.Dept}}
>>
login : DonJ
dept : IT
city : Las Vegas
title : CTO
samAccountName : DonJ
Name : DonJ
Department : IT

login : GregS
dept : Custodial
city : Denver
title : Janitor
samAccountName : GregS
Name : GregS
Department : Custodial

login : JeffH
dept : IT
city : Syracuse
title : Network Engineer
samAccountName : JeffH
Name : JeffH
Department : IT

We then created a hash table, which is the construct starting with @{ and ending
with }. Hash tables consist of one or more key=value pairs, and Select-Object
has been programmed to look for some specific keys, which we’ll provide to it.
 The first key Select-Object wants can be Name, N, Label, or L, and the value for
that key is the name of the property we want to create. In the first hash table, we
specified samAccountName, in the second, Name, and in the third, Department.
These correspond to the parameter names of New-ADUser.
We then created a hash table, which is the construct starting with @{ and ending
with }. Hash tables consist of one or more key=value pairs, and Select-Object
has been programmed to look for some specific keys, which we’ll provide to it.
 The first key Select-Object wants can be Name, N, Label, or L, and the value for
that key is the name of the property we want to create. In the first hash table, we
specified samAccountName, in the second, Name, and in the third, Department.
These correspond to the parameter names of New-ADUser.

TRY IT NOW Go ahead and create the CSV file that’s shown in figure 9.12.
Then try running the exact command we did above—you can type it exactly
as shown. 

What we’ve done is taken the contents of the CSV file—the output of Import-CSV—
and modified it, dynamically, in the pipeline. Our new output matches what
New-ADUser wants to see, so we can now create new users by running this command:
PS C:\> import-csv .\newusers.csv |
>> select-object -property *,
>> @{name='samAccountName';expression={$_.login}},
>> @{label='Name';expression={$_.login}},
>> @{n='Department';e={$_.Dept}} |
>> new-aduser
>>
The syntax might be a bit ugly, but this is an incredibly powerful technique. It’s also
usable in many other places in PowerShell, and you’ll see it again in upcoming chapters. You’ll even see it in the examples contained in PowerShell’s help files: Run Help
Select -Example and look for yourself.

9.6 Parenthetical commands
Sometimes, no matter how hard you try, you can’t make pipeline input work. For
example, consider the Get-WmiObject command. You’ll learn more about it in an
upcoming chapter, but for right now, look at the help for its -ComputerName property,
as shown in figure 9.13.
Figure 9.13 Reading the full help for Get-WmiObject

This parameter doesn’t accept computer names from the pipeline. How can we
retrieve names from someplace—like our text file, which contains one computer
name per line—and feed them to the command? The following won’t work:
PS C:\> get-content .\computers.txt | get-wmiobject -class win32_bios

The String objects produced by Get-Content won’t match the -computerName
parameter of Get-WmiObject. What can we do? Use parentheses:

PS C:\> Get-WmiObject -class Win32_BIOS -ComputerName (Get-Content .\comput
ers.txt)
Think back to high school algebra class, and you’ll recall that parentheses mean “do
this first.” That’s what PowerShell does: it runs the parenthetical command first.
The results of that command—in this case, a bunch of String objects—are fed to the
parameter. Because -ComputerName happens to want a bunch of String objects, the
command works.

TRY IT NOW If you have a couple of computers with which you can test this,
go ahead and try that command. Put the correct computer names or IP
addresses into your own Computers.txt file. This will work best for computers
all in the same domain, because permissions will be taken care of more easily
in that environment.

The parenthetical command trick is powerful because it doesn’t rely on pipeline
parameter binding at all—it takes objects and sticks them right into the parameter.
But the technique doesn’t work if your parenthetical command isn’t generating the
exact type of object that the parameter expects, which means sometimes you’ll have to
manipulate things a bit. Let’s look at how.

9.7 Extracting the value from a single property
Earlier in this chapter, we showed you an example of using parentheses to execute
Get-Content, feeding its output to the parameter of another cmdlet:
Get-Service -computerName (Get-Content names.txt)
Rather than getting your computer names from a static text file, you might want to
query them from Active Directory. With the ActiveDirectory module (available on a
Windows Server 2008 R2 or later domain controller, and installable with the Remote
Server Administration Tools, or RSAT), you could query all of your domain controllers:
get-adcomputer -filter * -searchbase "ou=domain controllers,
➥dc=company,dc=pri"

Could you use the same parentheses trick to feed computer names to Get-Service?
For example, would this work?
Get-Service -computerName (Get-ADComputer -filter *
➥-searchBase "ou=domain controllers,dc=company,dc=pri")

    Above and beyond
    If you don’t have a domain controller handy, that’s okay—we’ll quickly tell you what
    you need to know about the Get-ADComputer command.
    First, it’s contained in a module named ActiveDirectory. As we already mentioned,
    that module installs on any Windows Server 2008 R2 or later domain controller, and
    it’s available in the RSAT to install on a client computer that belongs to a domain.
    Second, the command—as you might expect—retrieves computer objects from the
    domain.
    Third, it has two useful parameters. -Filter * will retrieve all computers, and you
    could specify other filter criteria to limit the results, such as specifying a single computer name. The -SearchBase parameter tells the command where to start looking
    for computers; in this example, we’re having it start in the Domain Controllers organizational unit (OU) of the Company.com domain:
    get-adcomputer -filter * -searchbase "ou=domain controllers,
    ➥dc=company,dc=pri"
    Fourth, computer objects have a Name property, which contains the computers’ host
    name.
    We realize that throwing this kind of command at you—which, depending on your lab
    environment, you might not have access to—might be a bit unfair. But it’s an incredibly useful command for the scenarios we’re looking at, and it’s one you’d definitely
    want to use in a production environment. Provided you can keep the preceding four
    facts in mind, you should be fine for this chapter.
    
Sadly, it won’t. Look at the help for Get-Service, and you’ll see that the -computerName
parameter expects String values.
    Run this instead:
    get-adcomputer -filter * -searchbase "ou=domain controllers,
    ➥dc=company,dc=pri" | gm
Get-Member reveals that Get-ADComputer is producing objects of the type ADComputer.
Those aren’t String objects, so -computerName won’t know what to do with them. But
the ADComputer objects do have a Name property. What you need to do is extract the
values of the objects’ Name properties, and feed those values, which are computer
names, to the -ComputerName parameter.
    
TIP This is an important fact about PowerShell, and if you’re a bit lost right
now, STOP and reread the preceding paragraphs. Get-ADComputer produces
objects of the type ADComputer; Get-Member proves it. The -ComputerName
parameter of Get-Service can’t accept an ADComputer object; it accepts only
String objects, as shown in its help file. Therefore, that parenthetical command won’t work as written.

Once again, the Select-Object cmdlet can rescue you, because it includes an
-expandProperty parameter, which accepts a property name. It will take that property,
extract its values, and return those values as the output of Select-Object. Consider this
example:
get-adcomputer -filter * -searchbase "ou=domain controllers,
➥dc=company,dc=pri" | Select-Object -expand name
You should get a simple list of computer names. Those can be fed to the -computerName
parameter of Get-Service (or any other cmdlet that has a -computerName parameter):
Get-Service -computerName (get-adcomputer -filter *
➥-searchbase "ou=domain controllers,dc=company,dc=pri" |
➥Select-Object -expand name)
TIP Once again, this is an important concept. Normally, a command like
Select-Object -Property Name produces objects that happen to only have
a Name property, because that’s all we specified. The -computerName parameter doesn’t want some random object that has a Name property; it wants a
String, which is a much simpler value. -Expand Name goes into the Name
property and extracts its values, resulting in simple strings being returned
from the command.
Again, this is a cool trick that makes it possible to combine an even wider variety of commands with each other, saving you typing and making PowerShell do more of the work.
 Now that you’ve seen all that coolness with Get-ADComputer, let’s look at a similar
example using commands you should have access to. We’re assuming you’re running
the latest version of Windows, but for this example you don’t need to be in a domain,
or have access to a domain controller or even to a server OS. We’re going to stick with
the general theme of “getting computer names” because that’s such a common production need.

Figure 9.14 Make sure you can
import your CSV file using
Import-CSV and get results
similar to those shown here.

Now let’s say that you want to get a list of running processes from each of these computers. If you examine the help for Get-Process, as shown in figure 9.15, you’ll see that
its -computerName parameter does accept pipeline input ByPropertyName. It expects its
input to be objects of the type String. We’re not going to fuss around with pipeline
input, though; we’re going to focus on property extraction. The relevant information
in the help file is the fact that -ComputerName needs one or more String objects.

Figure 9.15 Verifying the data type needed by the -ComputerName parameter

Back to basics: start by seeing what Command A produces by piping it to Get-Member.
Figure 9.16 shows the results.
Import-CSV’s PSCustomObject output isn’t a String, so the following won’t work:
PS C:\> Get-Process -computerName (import-csv .\computers.csv)
Let’s try selecting the HostName field from the CSV, and see what that produces. You
should see what’s shown in figure 9.17.

in powershell type: 
Import-Csv .\computers.csv

Figure 9.16 Import-CSV produces objects of the type PSCustomObject.
Import-Csv .\computers.csv | gm

Figure 9.17 Selecting a single property still gives you a PSCustomObject.
Import-Csv .\computers.csv | select -property hostname | gm

You’ve still got a PSCustomObject, but it has fewer properties than before. That’s
the point about Select-Object and its -Property parameter: it doesn’t change the
fact that you’re outputting an entire object.
 The -ComputerName parameter can’t accept a PSCustomObject, so this still won’t
work:
PS C:\> Get-Process -computerName (import-csv .\computers.csv |
select -property hostname)
This is where the -ExpandProperty parameter works. Again, let’s try it on its own and
see what it produces, as shown in figure 9.18.
 Because the Hostname property contained text strings, -ExpandProperty was able
to expand those values into plain String objects, which is what -ComputerName wants.
This means the following will work:
PS C:\> Get-Process -computerName (import-csv .\computers.csv |
select -expand hostname)

import-csv .\compuiters.csv | select -expand hostname | getmember

Figure 9.18 You finally have a String object as output!

This is a powerful technique. It can be a little hard to grasp at first, but understanding
that a property is kind of like a box can help. With Select -Property, you’re deciding
what boxes you want, but you’ve still got boxes. With Select -ExpandProperty,
you’re extracting the contents of the box, and getting rid of the box entirely. You’re
left with the contents.



9.8 Lab
NOTE For this lab, you’ll need any computer running PowerShell v3.
Once again, we’ve covered a lot of important concepts in a short amount of time. The
best way to cement your new knowledge is to put it to immediate use. We recommend
doing the following tasks in order, because they build on each other to help remind
you what you’ve learned and to help you find practical ways to use that knowledge.
 To make this a bit trickier, we’re going to force you to consider the
Get-ADComputer command. Any Windows Server 2008 R2 or later domain controller
has this command installed, but you don’t need one. You only need to know three
things:
 The Get-ADComputer command has a -filter parameter; running
Get-ADComputer -filter * will retrieve all computer objects in the domain.
 Domain computer objects have a Name property that contains the computer’s
host name.
 Domain computer objects have the TypeName ADComputer, which means
Get-ADComputer produces objects of the type ADComputer.
That’s all you should need to know. With that in mind, complete these tasks:
NOTE You’re not being asked to run these commands. Instead, you’re being
asked if these commands will function or not, and why. You’ve been told how
Get-ADComputer works, and what it produces; you can read the help to discover what other commands expect and accept.
1 Would the following command work to retrieve a list of installed hotfixes from
all computers in the specified domain? Why or why not? Write out an explanation, similar to the ones we provided earlier in this chapter.
Get-Hotfix -computerName (get-adcomputer -filter * | Select-Object -expand name)
    This should work, because the nested Get-ADComputer expression will return a collection of computer names and the -Computername parameter can accpt an array of values 
    
2 Would this alternative command work to retrieve the list of hotfixes from the
same computers? Why or why not? Write out an explanation, similar to the ones
we provided earlier in this chapter.
get-adcomputer -filter * | Get-HotFix
    This won't work, because Get-HotFix doesn't accept any parameters by value. It will acccept -Computername property. This propery can be bond to the Computername parameter in Get-Hotfix because it accepts pipeline binding by property name.
    
3 Would this third version of the command work to retrieve the list of hotfixes
from the domain computers? Why or why not? Write out an explanation, similar to the ones we provided earlier in this chapter.
get-adcomputer -filter * |
Select-Object @{l='computername';e={$_.name}} | Get-Hotfix
    This should work. The first part of the expression is writing a custom object to the popeline that has a omputername property.  This property can be bound to the Computername parameter in Get-Hotfix because it acceots oipeline binding by property name.
4.Write a command that uses pipeline parameter binding to retrieve a list of running processes from every computer in an Active Directory (AD) domain. Don’t use parentheses.
    Get-ADComputer -filter * | -> Select-Object @{n='computername';e={$_.name}} | Get-Process
    
5 Write a command that retrieves a list of installed services from every computer
in an AD domain. Don’t use pipeline input; instead use a parenthetical command (a command in parentheses).
    Get-Service -Computername (get-adcomputer -filter * | Select-Object -expand property name)
    

6 Sometimes Microsoft forgets to add a pipeline parameter binding to a cmdlet.
For example, would the following command work to retrieve information from
every computer in the domain? Write out an explanation, similar to the ones we
provided earlier in this chapter.  get-adcomputer -filter * | Select-Object @{l='computername';e={$_.name}} | Get-WmiObject -class Win32_BIOS
    This will not work. The Computername parameter in Get-WMIObject doesn't take any pipeline binding.


9.9 Further exploration
We find that many students have difficulty embracing this pipeline input concept,
mainly because it’s so abstract. If you find yourself in that situation, head to MoreLunches.com. Find this book’s cover image or name, and click on it. Scroll to the
Downloads section, and download the Pipeline Input Workbook. Print as many copies as you like, grab a pencil, and start using it to walk through examples like
Get-Service | Stop-Service. The workbook provides step-by-step instructions for
working through the entire pipeline input process.
