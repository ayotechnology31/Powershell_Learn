r. What makes PowerShell special isn’t the way it runs commands,
but rather the way it allows multiple commands to be connected to each other in
powerful, one-line sequences.

6.1 Connect one command to another: less work for you
PowerShell connects commands to each other using something called a pipeline.
The pipeline provides a way for one command to pass, or pipe, its output to another
command, allowing that second command to have something to work with.

  You’ve already seen this in action in commands such as Dir | More. You’re piping the output of the Dir command into the More command;
You’re piping the output of the Dir command into the More command; the More command
takes that directory listing and displays it one page at a time.
. PowerShell takes that
same piping concept and extends it to greater effect. 

6.2 Exporting to a CSV or an XML file
Run a simple command. Here are a few suggestions:
 Get-Process (or Ps)
 Get-Service (or Gsv)
 Get-EventLog Security -newest 100

6.2.1 Exporting to CSV
Exporting to a file is where the pipeline and a second command come in handy:
Get-Process | Export-CSV procs.csv
Notepad procs.csv
Import-CSV procs.csv


6.2.2 Exporting to XML
 Export-CliXML  Extensible Markup Language
(XML) file. CliXML is unique to PowerShell, but any program capable of understanding XML can read it
 You’ll also have a matching Import-CliXML cmdlet. Both the
import and export cmdlets (such as Import-CSV and Export-CSV) expect a filename
as a mandatory parameter

6.2.3 Comparing files
Both CSV and CliXML files can be useful for persisting snapshots of information, sharing those snapshots with others, and reviewing those snapshots at a later time. In fact,
Compare-Object has a great way of using them. It has an alias, Diff, which we’ll use.
 First, run help diff and read the help for this cmdlet. We want you to pay attention to three parameters in particular: -ReferenceObject, -DifferenceObject, and
-Property.

We prefer using CliXML, rather than CSV, for comparisons like this, because CliXML
can hold more information than a flat CSV file. You’d then transport that XML file to
the difference computer, and run this command:
Diff -reference (Import-CliXML reference.xml)
➥-difference (Get-Process) -property Name

Because the previous step is a bit tricky, we’ll explain what’s happening:
 As in math, parentheses in PowerShell control the order of execution. In the
previous example, they force Import-CliXML and Get-Process to run before
Diff runs. 

6.3 Piping to a file or a printer
Whenever you have nicely formatted output—like the tables generated by
Get-Service or Get-Process—you may want to preserve that in a file, or even on
paper. Normally, cmdlet output is directed to the screen, which PowerShell refers to as
the host, but you can change where that output goes. In fact, we’ve already showed you
one way to do so:
Dir > DirectoryList.txt
The > character is a shortcut added to PowerShell to provide syntactic compatibility
with the older Cmd.exe shell. In reality, when you run that command, PowerShell
does the following under the hood:
Dir | Out-File DirectoryList.txt
This Dir, is really running this, Dir | Out-Default.  Out-Default does nothing more than direct content to
Out-Host, which means you’re running this,
Dir | Out-Default | Out-Host

Time to investigate other Out- cmdlets. To get started, try using
the Help command and wildcards such as Help Out*. Or  Get-Command Out*.
 Or, you could
specify the -verb parameter: Get-Command -verb Out

. If you do have those installed, try running Get-Service |
Out-GridView to see what happens. Out-Null and Out-String have specific uses that
we won’t get into right now, but you’re welcome to read their help files and look at the
examples included in those files.

6.4 Converting to HTML
Want to produce HTML reports? Easy: pipe your command to ConvertTo-HTML. 
In the PowerShell world, the verb Export implies that you’re taking data, converting it
to some other format, and saving that other format in some kind of storage, such as a
file. The verb ConvertTo implies only a portion of that process: the conversion to a different format, but not saving it into a file

TRY IT NOW If you can think of a way, go ahead and try it before you read on.
This command would do the trick:
Get-Service | ConvertTo-HTML | Out-File services.html

6.5 Using cmdlets that modify the system: killing
processes and stopping services
Exporting and converting aren’t the only reasons you might want to connect two commands together. 
For example, consider—but please do not run—this command:
Get-Process | Stop-Process
This will crash your computer!!! DO NOT RUN

Typically, you’d specify the name of a specific process
rather than trying to stop them all:
Get-Process -name Notepad | Stop-Process
Services offer something similar: the output from Get-Service can be piped to cmdlets
like Stop-Service, Start-Service, Set-Service, and so forth.


6.6 Common points of confusion
One common point of confusion in PowerShell revolves around the Export-CSV and
Export-CliXML commands. Both of these commands, technically speaking, create text
files. That is, the output of either command can be viewed in Notepad, as shown in
figure 6.2. But you have to admit that the text is definitely in a special kind of format—either in comma-separated values or XML.

 The confusion tends to set in when someone is asked to read these files back into
the shell. Do you use Get-Content (or its aliases, Type or Cat)? For example, suppose
you did this:
PS C:\> get-eventlog -LogName security -newest 5 | export-csv events.csv
Now, try reading that back in by using Get-Content:
PS C:\> Get-Content .\events.csv
We truncated the preceding output, but there’s a lot more of the same. Looks like garbage, right? You’re looking at the raw CSV data.
Much nicer, right? The Import- cmdlets pay attention to what’s in the file, attempt
to interpret it, and create a display that looks more like the output of the original
command (Get-EventLog, in this case). Typically then, if you create a file with
Export-CSV, you’ll read it by using Import-CSV. If you create it by using
Export-CliXML, you’ll generally read it by using Import-CliXML. By using these commands in pairs, you’ll get better results. Use Get-Content only when you’re reading in
a text file and don’t want PowerShell attempting to parse the data—that is, when you
want to work with the raw text.


6.7 Lab
NOTE For this lab, you’ll need any computer running PowerShell v3.
We’ve kept this chapter’s text slightly shorter because some of the examples we
showed you probably took a bit longer to complete, and because we want you to spend
more time completing the following hands-on exercises. If you haven’t already completed all of the “Try it now” tasks in the chapter, we strongly recommend that you do
so before tackling these tasks:
1. Create two similar, but different. text files. Try compaing them by using Diff. Run somethibg like this: Diff -reference (Get-Content File1.txt) -difference (Get-Content File2.txt). If the files have only one line of text that's the different, the command should work. 
    PS C:\> "I am the walrus" | out-file file1.text
    PS C:\> "I am a believer" | out-file file1.text
    PS:C:\> $f1-get-content .\file1.txt
    PS:C:\> $f2-get-content .\file2.txt
    PS C:\> diff $f1 $f2
    
    InputObject      Side Indicator
    ---------             ----------
    I'm a believer        =>
    I am the walrus    <=
    
2 What happens if you run Get-Service | Export-CSV services.csv | Out-File
from the console? Why does that happen?
    If you don't specify a filename with Out-File, you'll get an error.
    But even if youdo, Out-File won't do anything because the file is created by Export-CSV.
    
3 Apart from getting one or more services and piping them to Stop-Service,
what other means does Stop-Service provide for you to specify the service or
services you want to stop? Is it possible to stop a service without using
Get-Service at all?
    Stop-Service can accept one or more names as parameter vlue for the parameter. For example, you could run this: Stop-Service spooler
    
4 What if you want to create a pipe-delimited file instead of a comma-separated
(CSV) file? You would still use the Export-CSV command, but what parameters
would you specify?
    get-service | Export-Csv services.csv -Delimiter *|*
    
    
5 Is there a way to eliminate the # comment line from the top of an exported CSV
file? That line normally contains type information, but what if you want to omit
that from a particular file?
    Use the -NoTypeInformation parameter with Export-CSV.

6 Export-CliXML and Export-CSV both modify the system because they can create and overwrite files. What parameter would prevent them from overwriting
an existing file? What parameter would ask you if you were sure before proceeding to write the output file?
    get-service | Export-Csv services.csv services.csv -noclobber
    get-service | Export-Csv services.csv services.csv -confirm
    
7 Windows maintains several regional settings, which include a default list separator. On U.S. systems, that separator is a comma. How can you tell Export-CSV to
use the system’s default separator, rather than a comma?
get-service | Export-Csv services.csv services.csv -UseCulture



TRY IT NOW After you’ve completed this lab, try to complete Review Lab 1,
which you’ll find in appendix A of this book.

