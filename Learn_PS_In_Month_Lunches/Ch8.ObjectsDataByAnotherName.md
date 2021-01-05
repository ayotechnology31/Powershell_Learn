We’re going to do something a little different in this chapter. We find that PowerShell’s use of objects can be one of its most confusing elements, but at the same
time it’s also one of the shell’s most critical concepts, affecting everything you do in
the shell. We’ve tried different explanations over the years, and we’ve settled on a
couple that each work well for distinctly different audiences. If you have some programming experience and you’re comfortable with the concept of objects, we want
you to skip to section 8.2. If you don’t have a programming background, and
haven’t programmed or scripted with objects before, start with section 8.1 and read
straight through the chapter.

Take a second to run Get-Process in PowerShell. You should see a table with several columns, but those columns barely scratch the surface of the wealth of information available about processes. Each process object also has a machine name, a
main window handle, a maximum working set size, an exit code and time, processor affinity information, and a great deal more. In fact, you’ll find more than 60
pieces of information associated with a process. Why does PowerShell show so few
of them?
 The simple fact is that most of the things PowerShell can access offer more information than will comfortably fit on the screen. When you run any command, such
Licensed to <pedbro@gmail.com>
86 CHAPTER 8 Objects: data by another name
as Get-Process, Get-Service, Get-EventLog, or anything, PowerShell constructs—
entirely in memory—a table that contains all of the information about those items. In
the case of Get-Process, that table consists of something like 67 columns, with one
row for each process that’s running on your computer. Each column contains a bit of
information, such as virtual memory, CPU utilization, process name, process ID, and
so on. Then, PowerShell looks to see if you’ve specified which of those columns you
want to view. If you haven’t (and we haven’t shown you how, yet), then the shell looks
up a configuration file provided by Microsoft and displays only those table columns
that Microsoft thought you’d want to see.
 One way to see all of the columns is to use ConvertTo-HTML:
Get-Process | ConvertTo-HTML | Out-File processes.html

That cmdlet doesn’t bother filtering down the columns. Instead, it produces an HTML
file that contains all of them. That’s one way to see the entire table.
 In addition to all of those columns of information, each table row also has some
actions associated with it. Those actions include what the operating system can do to,
or with, the process listed in that table row. For example, the operating system can
close a process, kill it, refresh its information, or wait for the process to exit, among
other things.
 Any time you run a command that produces output, that output takes the form of
a table in memory. When you pipe output from one command to another, like this
the entire table is passed through the pipeline. The table isn’t filtered down to a
smaller number of columns until every command has run. 

Now for some terminology changes. PowerShell doesn’t refer to this in-memory
table as a “table.” Instead, it uses these terms:
 Object—This is what we’ve been calling a “table row.” It represents a single
thing, like a single process or a single service.
 Property—This is what we called a “table column.” It represents one piece of
information about an object, like a process name, process ID, or service status.
 Method—This is what we called an “action.” A method is related to a single
object and makes that object do something, like killing a process or starting a
service.
 Collection—This is the entire set of objects, or what we’ve been calling a “table.”
If you ever find the following discussion on objects to be confusing, refer back to this
four-point list. Always imagine a collection of objects as being a big in-memory table of
information, with properties as the columns and individual objects as the rows. 

8.2 Why PowerShell uses objects
One of the reasons why PowerShell uses objects to represent data is that, well, you have
to represent data somehow, right? PowerShell could have stored that data in a format
like XML, or perhaps its creators could have decided to use plain-text tables. But they
had some specific reasons why they didn’t take that route.
 The first reason is that Windows itself is an object-oriented operating system—or at
least, most of the software that runs on Windows is object oriented. Choosing to structure data as a set of objects is easy, because most of the operating system lends itself to
those structures.
 Another reason to use objects is because they ultimately make things easier on you
and give you more power and flexibility. For the moment, let’s pretend that PowerShell
doesn’t produce objects as the output of its commands. Instead, it produces simple
text tables, which is what you probably thought it was doing in the first place. When
you run a command like Get-Process, you’re getting formatted text as the output:

PS C:\Users\cookie> get-process

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName             
-------  ------    -----      -----     ------     --  -- -----------             
    248      15     4248       3192       0.36   9588   1 ApplicationFrameHost    
    166       8     7192       6760      61.48    720   1 bash                    
    162       9     1780       2296       0.19   4892   1 chrome                  
    354      27    94564      37140      30.53   7708   1 chrome                  
    214      14    11596      21028       0.09   8360   1 chrome                  
    385      26    36620      40576   4,237.72  11024   1 chrome                  
   1181      44    59668      77824   3,869.19  11064   1 chrome                  
    355      38    82272      83268     283.03  11676   1 chrome                  
    289      20    16192      19912      21.95  13220   1 chrome                  
    202      13     6312       4988       0.39  13264   1 chrome                  
    137       8     1616       1100              2940   0 coherence         
    What if you wanted to do something else with this information? Perhaps you want to
    make a change to all of the processes running Conhost. To do this, you’d have to filter
    the list down a bit. In a Unix or Linux shell, you’d use a command like Grep, telling it,
    “Look at this text list for me. Keep only those rows where columns 58–64 contain the
    characters ‘conhost.’ Delete all of the other rows.” The resulting list would contain
    only those processes you specified:
    Handles NPM(K) PM(K) WS(K) VM(M) CPU(s) Id ProcessName
    ------- ------ ----- ----- ----- ------ -- -----------
     39 5 1876 4340 52 11.33 1920 conhost
     31 4 792 2260 22 0.00 2460 conhost
     29 4 828 2284 41 0.25 3192 conhost
     
     You’d then pipe that text to another command, perhaps telling it to extract the process ID from the list. “Go through this and get the characters from columns 52–56, but
     drop the first two (header) rows.” The result might be this:
     1920
     2460
     3192
     Finally, you’d pipe that text to yet another command, asking it to kill the processes (or
     whatever else you were trying to do) represented by those ID numbers.
      This is, in fact, exactly how Unix and Linux administrators work. They spend a lot
     of time learning how to get better at parsing text, using tools like Grep, Awk, and Sed,
     and becoming proficient in the use of regular expressions. Going through this learning process makes it easier for them to define the text patterns they want their computer to look for. Unix and Linux folks like programming languages like Perl because
     those languages contain rich text-parsing and text-manipulation functions.
      But this text-based approach does present some problems:
      You can spend more time messing around with text than doing your real job.
      If the output of a command changes—say, moving the ProcessName column to
     the start of the table—then you have to rewrite all of your commands, because
     they’re all dependent on things like column positions.
      You have to become proficient in languages and tools that parse text. Not
     because your job involves parsing text, but because parsing text is a means to
     an end.
     PowerShell’s use of objects helps to remove all of that text-manipulation overhead
     

8.3 Discovering objects: Get-Member
If objects are like a giant table in memory, and PowerShell only shows you a portion of
that table on the screen, how can you see what else you have to work with? If you’re
thinking that you should use the Help command, we’re glad, because we’ve certainly
been pushing that down your throat in the previous few chapters. Unfortunately,
you’d be wrong.
 The help system only documents background concepts (in the form of the “about”
help topics) and command syntax. To learn more about an object, you use a different
command: Get-Member. You should become comfortable using this command—so
much so, in fact, that you start looking for a shorter way to type it. We’ll give you that
right now: the alias Gm.
 You can use Gm after any cmdlet that normally produces some output. For example,
you already know that running Get-Process produces some output on the screen.
You can pipe it to Gm:
Get-Process | Gm
Whenever a cmdlet produces a collection of objects, as Get-Process does, the entire
collection remains accessible until the end of the pipeline. It’s not until every command has run that PowerShell filters down the columns of information to be displayed and creates the final text output you see. Therefore, in the preceding example, Gm has
complete access to all of the process objects’ properties and methods, because they
haven’t been filtered down for display yet. Gm looks at each object and constructs a list
of the objects’ properties and methods. It looks a bit like this:
PS C:\> get-process | gm
 TypeName: System.Diagnostics.Process
Name MemberType Definition
---- ---------- ----------
Handles AliasProperty Handles = Handlecount
Name AliasProperty Name = ProcessName
NPM AliasProperty NPM = NonpagedSystemMemo...
PM AliasProperty PM = PagedMemorySize
VM AliasProperty VM = VirtualMemorySize
WS AliasProperty WS = WorkingSet
Disposed Event System.EventHandler Disp...
ErrorDataReceived Event System.Diagnostics.DataR...
Exited Event System.EventHandler Exit...
OutputDataReceived Event System.Diagnostics.DataR...
BeginErrorReadLine Method System.Void BeginErrorRe...
BeginOutputReadLine Method System.Void BeginOutputR...
CancelErrorRead Method System.Void CancelErrorR...
CancelOutputRead Method System.Void CancelOutput...
We’ve trimmed the preceding list a bit because it’s long, but hopefully you get the
idea.
TRY IT NOW Don’t take our word for it. This is the perfect time to follow  along
and run the same commands we do, in order to see their complete output.
By the way, it may interest you to know that all of the properties, methods, and other
things attached to an object are collectively called its members, as if the object itself
were a country club and all of these properties and methods belonged to the club.
That’s where Get-Member takes its name from: it’s getting a list of the objects’ members. But remember, because the PowerShell convention is to use singular nouns, the
cmdlet name is Get-Member, not “Get-Members.” 
IMPORTANT It’s easy to overlook, but pay attention to the first line of output
from Get-Member. It’s the TypeName, which is the unique name assigned to
that particular type of object. It may seem unimportant now—after all, who
cares what it’s named? But it’s going to become crucial in the next chapter.


8.4 Object attributes, or “properties”
When you examine the output of Gm, you’ll notice several different kinds of properties:
 ScriptProperty
 Property
 NoteProperty
 AliasProperty

Above and beyond
Normally, objects in the .NET Framework—which is where all of PowerShell’s objects
come from—have only “properties.” PowerShell dynamically adds the other stuff:
ScriptProperty, NoteProperty, AliasProperty, and so on. If you happen to look
up an object type in Microsoft’s MSDN documentation (you can plug the object’s
TypeName into your favorite search engine to find the MSDN page), you won’t see
these extra properties.
PowerShell has an Extensible Type System (ETS) that’s responsible for adding these
last-minute properties. Why does it do this? In some cases, it’s to make objects more
consistent, such as adding a Name property to objects that natively only have something like ProcessName (that’s what an AliasProperty is for). Sometimes it’s to
expose information that’s deeply buried in the object (process objects have a few
ScriptProperties that do this).
Once you’re in PowerShell, these properties all behave the same way. But don’t be
surprised when they don’t show up on the official documentation page: the shell adds
these extras, often to make your life easier.

For your purposes, these properties are all the same. The only difference is in how the
properties were originally created, but that’s not something you need to worry about.
To you, they’re all “properties,” and you’ll use them the same way.
 A property always contains a value. For example, the value of a process object’s ID
property might be 1234, and the Name property of that object might have a value of
Notepad. Properties describe something about the object: its status, its ID, its name,
and so on. In PowerShell, properties are often read-only, meaning you can’t change
the name of a service by assigning a new value to its Name property. But you can
retrieve the name of a service by reading its Name property. We’d estimate that 90 percent of what you’ll do in PowerShell will involve properties.


8.5 Object actions, or “methods”
Many objects support one or more methods, which, as we mentioned earlier, are
actions that you can direct the object to take. A process object has a Kill method,
which terminates the process. Some methods require one or more input arguments
that provide additional details for that particular action, but this early in your PowerShell education you won’t be running into any of those. In fact, you may spend
months or even years working with PowerShell and never need to execute a single
object method. That’s because many of those actions are also provided by cmdlets.
Get-Process -Name Notepad | Stop-Process
You could also accomplish that by using a single cmdlet:
Stop-Process -name Notepad

Our focus with this book is entirely on using PowerShell cmdlets to accomplish tasks.
They provide the easiest, most administrator-centric, most task-focused way of accomplishing things. Using methods starts to edge into .NET Framework programming,
which can be more complicated and can require a lot more background information.
For that reason, you’ll rarely—if ever—see us execute an object method in this book.
In fact, our general philosophy at this point is, “if you can’t do it with a cmdlet, go
back and use the GUI.” You won’t feel that way for your entire career, we promise, but
for now it’s a good way to stay focused on the “PowerShell way” of doing things.

Above and beyond
You don’t need to know about them at this stage in your PowerShell education, but
in addition to properties and methods, objects can also have events. An event is an
object’s way of notifying you that something happened to it. A process object, for
example, can trigger its Exited event when the process ends. You can attach your
own commands to those events, so that, for example, an email gets sent when a process exits. Working with events in this fashion is an advanced topic, and beyond the
scope of this book.

8.6 Sorting objects
Most PowerShell cmdlets produce objects in a deterministic fashion, which means
that they tend to produce objects in the same order every time you run the command.
Both services and processes, for example, are listed in alphabetical order by name.
Event log entries tend to come out in chronological order. What if we want to change
that?
 For example, suppose we want to display a list of processes, with the biggest consumers of virtual memory (VM) at the top of the list, and the smallest consumers at the
bottom. We would need to somehow reorder that list of objects based on the VM property. PowerShell provides a simple cmdlet, Sort-Object, which does exactly that:
Get-Process | Sort-Object -property VM

TRY IT NOW We’re hoping that you’ll follow along and run these same commands. We won’t be pasting the output into the book because these tables are
somewhat long, but you’ll get substantially the same thing on your screen if
you’re following along.
That command isn’t exactly what we wanted. It did sort on VM, but it did so in ascending order, with the largest values at the bottom of the list. Reading the help for
Sort-Object, we see that it has a -descending parameter that should reverse the sort
order. We also notice that the -property parameter is positional, so we don’t need to
type the parameter name. We’ll also tell you that Sort-Object has an alias, Sort, so
you can save yourself a bit of typing for the next try:
Get-Process | Sort VM -desc

We also abbreviated -descending to -desc, and we have the result we wanted. The
-property parameter accepts multiple values (which we’re sure you saw in the help
file, if you looked).
 In the event that two processes are using the same amount of virtual memory, we’d
like them sorted by process ID, and the following command will accomplish that:
Get-Process | Sort VM,ID -desc
As always, a comma-separated list is the way to pass multiple values to any parameter
that supports them.
 
 8.7 Selecting the properties you want 
 Another useful cmdlet is Select-Object. It accepts objects from the pipeline, and you
 can specify the properties that you’d like displayed. This enables you to access properties that are normally filtered out by PowerShell’s configuration rules, or to trim down
 the list to a few properties that interest you. This can be useful when piping objects to
 ConvertTo-HTML, because that cmdlet usually builds a table containing every property.
  Compare the results of these two commands:
 Get-Process | ConvertTo-HTML | Out-File test1.html
 Get-Process | Select-Object -property Name,ID,VM,PM |
 ➥Convert-ToHTML | Out-File test2.html
 TRY IT NOW Go ahead and run each of these commands separately, and then
 examine the resulting HTML files in Internet Explorer to see the differences.
 Take a look at the help for Select-Object (or you can use its alias, Select). The
 -property parameter appears to be positional, which means we could shorten that
 last command to:
 Get-Process | Select Name,ID,VM,PM | ConvertTo-HTML | Out-File test3.html
 Spend some time experimenting with Select-Object. In fact, try variations of the following command, which allows the output to appear on the screen:
 Get-Process | Select Name,ID,VM,PM
 Try adding and removing different process object properties from that list and reviewing the results. How many properties can you specify and still get a table as the output? How many properties force PowerShell to format the output as a list rather than
 as a table?
 
 Above and beyond
 Select-Object also has -First and -Last parameters, which let you keep a subset of the objects in the pipeline. For example, Get-Process | Select -First 10
 would keep the first ten objects. There’s no criteria involved, like keeping certain processes; it’s merely grabbing the first (or last) ten.
 
 CAUTION People often get mixed up about two PowerShell commands:
 Select-Object and Where-Object, which you haven’t seen yet.
 Select-Object is used to choose the properties (or columns) you want to
 see, and it can also select an arbitrary subset of output rows (using -First
 and -Last). Where-Object removes, or filters, objects out of the pipeline
 based on some criteria you specify. 

8.8 Objects until the end
The PowerShell pipeline always contains objects until the last command has been executed. At that time, PowerShell looks to see what objects are in the pipeline, and then
looks at its various configuration files to see which properties to use to construct the
onscreen display. It also decides whether that display will be a table or a list, based on
some internal rules and on its configuration files. (We’ll explain more about those
rules and configurations, and how you can modify them, in an upcoming chapter.)
 An important fact is that the pipeline can contain many different kinds of objects
over the course of a single command line. For the next few examples, we’re going to
take a single command line and physically type it so that only one command appears
on a single line of text. That’ll make it a bit easier to explain what we’re talking about.
 Here’s the first one:
Get-Process |
Sort-Object VM -descending |
Out-File c:\procs.txt

In this example, you start by running Get-Process, which puts process objects into
the pipeline. The next command is Sort-Object. That doesn’t change what’s in the
pipeline; it changes only the order of the objects, so at the end of Sort-Object, the
pipeline still contains processes. The last command is Out-File. Here, PowerShell has
to produce output, so it takes whatever’s in the pipeline—processes—and formats
them according to its internal rule set. The results go into the specified file.
 Next up is a more complicated example

TRY IT NOW Try running this three-cmdlet command line, keeping in mind
that you should type the whole thing on a single line. Notice how the output
is different from the normal output of Get-Process?

Get-Process |
Sort-Object VM -descending |
Select-Object Name,ID,VM

This starts off in the same way. Get-Process puts process objects into the pipeline.
Those go to Sort-Object, which sorts them and puts the same process objects into the
pipeline. But Select-Object works a bit differently. A process object always has the
exact same members. In order to trim down the list of properties, Select-Object
can’t remove the properties you don’t want, because the result wouldn’t be a process
object anymore. Instead, Select-Object creates a new kind of custom object called a
PSObject. It copies over the properties you do want from the process, resulting in a
custom object being placed into the pipeline.

TRY IT NOW Try running this three-cmdlet command line, keeping in mind
that you should type the whole thing on a single line. Notice how the output
is different from the normal output of Get-Process?

When PowerShell sees that it’s reached the end of the command line, it has to decide
how to lay out the text output. Because there are no longer any process objects in the
pipeline, PowerShell won’t use the default rules and configurations that apply to process objects. Instead, it looks for rules and configurations for a PSObject, which is
what the pipeline now contains. Microsoft didn’t provide any rules or configurations
for PSObjects, because they’re meant to be used for custom output. Instead, PowerShell takes its best guess and produces a table, on the theory that those three pieces of
information probably will still fit in a table. The table isn’t as nicely laid out as the normal output of Get-Process, though, because the shell lacks the additional configuration information needed to make a nicer-looking table.
 You can use Gm to see the different objects that wind up in the pipeline. Remember,
you can add Gm after any cmdlet that produces output:
Get-Process | Sort VM -descending | gm
Get-Process | Sort VM -descending | Select Name,ID,VM | gm
TRY IT NOW Try running those two command lines separately, and notice the
difference in the output.
Notice that, as part of the Gm output, PowerShell shows you the type name for the
object it saw in the pipeline. In the first case, that was a System.Diagnostics.Process
object, but in the second case the pipeline contains a different kind of object. Those
new “selected” objects only contained the three properties specified—Name, ID, and
VM—plus a couple of system-generated members.
 Even Gm produces objects and places them into the pipeline. After running Gm, the
pipeline no longer contains either process or the “selected” objects; it contains the type
of object produced by Gm: a Microsoft.PowerShell.Commands.MemberDefinition.
You can prove that by piping the output of Gm to Gm itself:
Get-Process | Gm | Gm
TRY IT NOW You’ll definitely want to try this, and think hard about it to make
sure it makes sense to you. You start with Get-Process, which puts process
objects into the pipeline. Those go to Gm, which analyzes them and produces its
own MemberDefinition objects. Those are then piped to Gm, which analyzes
them and produces output that lists the members of a MemberDefinition
object.
A key to mastering PowerShell is learning to keep track of what kind of object is in the
pipeline at any given point. Gm can help you do that, but sitting back and verbally walking yourself through the command line is also a good exercise that can help clear up
confusion.


8.9 Common points of confusion
Our classroom students tend to make a few common mistakes as they get started with
PowerShell. Most of these go away with a little experience, but we’ll direct your attention to them with the following list, to give you a chance to catch yourself if you
start heading down the wrong path.
 Remember that the PowerShell help files don’t contain information on objects’
properties. You’ll need to pipe the objects to Gm (Get-Member) to see a list of
properties.
 Remember that you can add Gm to the end of any pipeline that normally produces results. A command line like Get-Process -name Notepad | Stop-Process
doesn’t normally produce results, so tacking | Gm onto the end won’t produce
anything either.
 Pay attention to neat typing. Put a space on either side of every pipeline character, because your command lines should read like Get-Process | Gm and not
like Get-Process|Gm. That spacebar key is extra-large for a reason—use it.
 Remember that the pipeline can contain different types of objects at each step.
Think about what type of object is in the pipeline, and focus on what the next
command will do to that type of object.

8.10 Lab
This chapter has probably covered more, and more difficult, new concepts than any
chapter to this point. We hope we were able to make sense of it all, but these exercises
will help you cement what you’ve learned. See if you can complete all of the exercises,
and remember to supplement your learning with the companion videos and sample
solutions at MoreLunches.com. Some of these tasks will draw on skills you’ve learned
in previous chapters, to refresh your memory and keep you sharp.
1 Identify a cmdlet that will produce a random number.
Get-Random 

2 Identify a cmdlet that will display the current date and time.
Get-Date 

3 What type of object does the cmdlet from task #2 produce? (What is the type
name of the object produced by the cmdlet?)

4 Using the cmdlet from task #2 and Select-Object, display only the current day
of the week in a table like the following (caution: the output will right-align, so
make sure your PowerShell window doesn’t have a horizontal scroll bar):
DayOfWeek

---------
Monday

5 Identify a cmdlet that will display information about installed hotfixes.

6 Using the cmdlet from task #5, display a list of installed hotfixes. Sort the list by
the installation date, and display only the installation date, the user who
installed the hotfix, and the hotfix ID. Remember that the column headers
shown in a command’s default output aren’t necessarily the real property
names—you’ll need to look up the real property names to be sure.

7 Repeat task #6, but this time sort the results by the hotfix description, and
include the description, the hotfix ID, and the installation date. Put the results
into an HTML file.

8 Display a list of the 50 newest entries from the Security event log (you can use a
different log, such as System or Application, if your Security log is empty). Sort
the list with the oldest entries appearing first, and with entries made at the same
time sorted by their index. Display the index, time, and source for each entry.
Put this information into a text file (not an HTML file, but a plain text file). You
may be tempted to use Select-Object and its -first or -last parameters to
achieve this; don’t. There’s a better way. Also, avoid using Get-WinEvent for
now; a better cmdlet is available for this particular task.


