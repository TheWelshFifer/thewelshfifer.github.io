# Turning a PowerShell Script into a Tool

I will try and explain how easy it is to turn a simple script or cmdlet into a useful and powerful tool that you can use and share.

let's use the example of having a simple cmdlet to return if a server is up or not.  There is a default cmdlet that we can use for that called Test-NetConnection, we will base our example around this cmdlet and specify the parameter "CommonTCPPort" to check for RDP. (remote desktop!)



To test a single server you can type the following into a PowerShell prompt:

```powershell
PS C:\Users\JKelly> Test-NetConnection -ComputerName SERVER1 -CommonTCPPort RDP
```

This will then return the results:

```powershell
ComputerName     : SERVER1
RemoteAddress    : 10.22.120.240
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 10.22.202.86
TcpTestSucceeded : True
```

You can can see that the TcpTestSucceeded value is "True".  Great start! - this means that the server is up and will accept RDP connections.  Next let's turn this into more of a script that someone can easily edit and use for other servers.

---

## Adding Variables

Using a PowerShell editor create a new .ps1 file.  We will create a variable. A variable is just a storage mechanism for storing a value.  You can specify a variable by typing a dollar symbol followed by the name of your variable:

```powershell
$ServerName = "SERVER1"
```

Run this code and then type $ServerName at the PowerShell prompt.  This should return:

```powershell
PS C:\Users\JKelly> $ServerName = "SERVER1"
PS C:\Users\JKelly> $ServerName
SERVER1
```

We can now use our variable to pass the computer name to the Test-NetConnection cmdlet:

```powershell
$ServerName = "SERVER1"
Test-NetConnection -ComputerName $ServerName -CommonTCPPort RDP
```

This produces the output:

```powershell
ComputerName     : SERVER1
RemoteAddress    : 10.22.120.240
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 10.22.202.86
TcpTestSucceeded : True
```

In the above example, you can see we have exactly the same result but are using a variable to replace the hard-coded server name.

---

## Formating the Output

Next up, let's format the output, all we are interested in is if the server is available or not; let's add a IF ELSE statement to check the property "TcpTestSucceeded".  If the test reports back as true then text is written back to screen to say it's up, else, text is written back to screen to say it's down.

So building on our existing code we are going to wrap the Test-NetConnection cmdlet into a if statement, the formatting looks a bit more complex than it is so let's break it down first.

As we stated before, all we are interested in is knowing if the TcpTestSucceeded is true or false.  To return only this value, let's format our command a bit differently:

We will initially wrap our command in brackets and then append a '.' followed by the property we want.  In this case "TcpTestSucceeded".  So, our code will look like this:

```powershell
$ServerName = "SERVER1"
(Test-NetConnection -ComputerName $ServerName -CommonTCPPort RDP).TcpTestSucceeded
```

When we run this code the following output is returned:

```powershell
True
```

Looks rubbish but now we have something for our If statement to check!  So, let's go ahead and build our If statement.  A If statement is wrapped in brackets followed by "squiggly brackets!" {}  i.e. :

```powershell
If ($something){
    do-something
}
```

Think of the squiggly brackets as a then statement - i.e. If this THEN do everything between these squiggly brackets.

So let's add a If statement to our code:
> Remember our code was already in brackets so we will need another set of brackets for this to work.

```powershell
$ServerName = "SERVER1"
If ((Test-NetConnection -ComputerName $ServerName -CommonTCPPort RDP).TcpTestSucceeded) {
    Write-Host "$ServerName is up!"
}
```

Looking at our code, we are saying if the Test-NetConnection is true then use the write-host cmdlet to output to screen that the content of the $ServerName variable is up!  Running this code returns:

```powershell
SERVER1 is up!
```

Fantastic! - so, the obvious question, What happens if the server is down.  Let's test it.  We will change our variable to a server that we know is unavailable:

```powershell
$ServerName = "SERVER4"
If ((Test-NetConnection -ComputerName $ServerName -CommonTCPPort RDP).TcpTestSucceeded) {
    Write-Host "$ServerName is up!" -ForegroundColor Green
}
```

Running this code returns:

```powershell
WARNING: Name resolution of SERVER4 failed
```

That's useful output and we can see that our Write-Host cmdlet has not been used.  However we may want to do something else if our test fails so we can add an else statement.  The way we do this, is similar to the initial IF statement.  so extending our original IF example:

```powershell
If ($Something) {
    Do-Something
}
Else {
    Do-SomethingElse
}
```

So to be clear - the value within the brackets of the If statement is evaluated, if it's true then the code between the squiggly brackets of the If statement will be executed, if the value is false then the code between the squiggly brackets of the Else statement will be executed.

Let's now add the Else statement to our code:

```powershell
$ServerName = "SERVER4"
If ((Test-NetConnection -ComputerName $ServerName -CommonTCPPort RDP).TcpTestSucceeded) {
    Write-Host "$ServerName is up!"
}
Else{
    Write-Host "$ServerName is down!"
}
```

Save and run the code, and if the specified server is down the following result will be displayed:

```powershell
WARNING: Name resolution of SERVER4 failed
SERVER4 is down!
```

So we can now see the warning from the command is passed back along with our text to confirm the server is down.  So finally, testing the output with a server we know is up:

```powershell
PS C:\Users\JKelly> $ServerName = "SERVER1"
If ((Test-NetConnection -ComputerName $ServerName -CommonTCPPort RDP).TcpTestSucceeded) {
    Write-Host "$ServerName is up!"
}
Else{
    Write-Host "$ServerName is down!"
}
```

This produces the output:

```powershell
SERVER1 is up!
```

---

## Add Multiple Object Support

It would be very handy to support multiple computers,  To do this we can separate multiple values in our variable with commas:

```powershell
$ServerName = "SERVER1", "SERVER2"
```

Now we need to tell our script to process each of the objects in our variable.  To do this we will add in a ForEach loop.  This is just running your code for each object that's in your variable.  Again, the format of this is very similar to the If statement:

```powershell
$ServerName = "SERVER1", "SERVER2"
ForEach ($Server in $ServerName) {
    If ((Test-NetConnection -ComputerName $Server -CommonTCPPort RDP).TcpTestSucceeded) {
        Write-Host "$Server is up!" -ForegroundColor Green
    }
    Else{
        Write-Host "$Server is down!" -ForegroundColor Red
    }
}
```

Save and run your code and it will produce output similar to:

```powershell
SERVER1 is up!
SERVER2 is up!
```

Great progress so far, we have a script that will test multiple servers and report back to screen!

---

## Parameterise the Script

Now we are starting to have a handy script to quickly check multiple servers, we still have a problem that to change what servers are being checked, the script needs to be edited.  This could result in someone accidentally changing the code of our script.  We want to avoid that!  We can do this by using Parameters.

To add parameters to our script we add a "Param" section to the beginning of the script.  This will allow exactly the same functionality to be used but the variable values passed into the script rather than the script being edited.  The statement looks something like this:

```powershell
Param([string[]]$ServerName)
```

People often format Parameters differently, I personally split mine over different lines as I find it easier to read so the above parameter can be written as:

```powershell
Param(
  [string[]]$ServerName
)
```

This is really handy if you have multiple parameters to specify.  I won't go into the detail of parameters but there are loads of tutorials online, the code above will do the following:

1. Create a parameter called "ServerName"
2. Defining this as a text string.
3. Storing the value of the parameter in a variable called $ServerName.

Our code now looks like this:

```powershell
Param(
  [string[]]$ServerName
)

ForEach ($Server in $ServerName) {
    If ((Test-NetConnection -ComputerName $Server -CommonTCPPort RDP).TcpTestSucceeded) {
        Write-Host "$Server is up!" -ForegroundColor Green
    }
    Else{
        Write-Host "$Server is down!" -ForegroundColor Red
    }
}
```

Save your script and then execute it from the PowerShell prompt by entering something similar to:

```powershell
PS C:\Users\JKelly\desktop> .\ServerCheck.ps1 -ServerName "SERVER1","SERVER3"
SERVER1 is up!
SERVER3 is up!
```

We now have a tool that can be handed to anyone to use!

---

## Convert the Script into a Function

What can we do next?  Well let's turn this script into a function.  A function will turn this script into a cmdlet that can then be used in a script for a different purpose.  Certainly now moving into the realms of creating a tool!

A function statement is very straight-forward.  We simply wrap the code in a statement for example:

```powershell
Function My-Function {
    Write-Host "Code goes here"
}
```

If you were to save and execute that block of code you will see it's technically operating like a PowerShell function:

```powershell
PS C:\Users\JKelly> My-function
Code goes here
```

Obviously that's not very useful, so lets wrap our code in a function called Get-ServerStatus:

```powershell
Function Get-ServerStatus {
    Param(
        [string[]]$ServerName
    )
    foreach ($Server in $ServerName) {
        if ((Test-NetConnection -ComputerName $Server -CommonTCPPort RDP).TcpTestSucceeded) {
            Write-Host "$Server is up!" -ForegroundColor Green
        }
        else {
            Write-Host "$Server is down!" -ForegroundColor Red
        }
    }
}
```

We are ready to test.  Save and run the script and you will notice nothing happens, now you will be able to type the function name at the PowerShell command line:

```powershell
PS C:\Users\JKelly> Get-ServerStatus -ServerName SERVER1
SERVER1 is up!
```

---

## Mandatory Parameters

 So, you are hopefully now starting to see the benefit of creating functions, the next thing we could  add is ensuring that the parameter is mandatory.  If someone uses the function incorrectly and does not specify a computer name we want the script to fail and the user informed that the parameter is mandatory.

To do this we use a slightly more complex method of defining a parameter:

```powershell
[CmdletBinding()]
Param(
    [Parameter(Mandatory=$true)]
    [string[]]$ServerName
)
```

The first line here is allowing cmdlet binding, this allows the function to operate like a fully complied cmdlet and will provide access to other "default" parameters such as -Verbose etc.

Next we specify the Parameter is mandatory and finally, the type and name of the parameter is specified.  This results in our code now looking like this:

```powershell
Function Get-ServerStatus {
    [CmdletBinding()]
    Param(
        [Parameter(Mandatory=$true)]
        [string[]]$ServerName
    )
    foreach ($Server in $ServerName) {
        if ((Test-NetConnection -ComputerName $Server -CommonTCPPort RDP).TcpTestSucceeded) {
            Write-Host "$Server is up!" -ForegroundColor Green
        }
        else {
            Write-Host "$Server is down!" -ForegroundColor Red
        }
    }
}
```

When we save and run our script again, you will now be able to just type Get-ServerStatus but then be prompted for at least one server name:

```powershell
PS C:\Users\JKelly> Get-ServerStatus
cmdlet Get-ServerStatus at command pipeline position 1
Supply values for the following parameters:
ServerName[0]: SERVER1
ServerName[1]: SERVER3
ServerName[2]: SERVER2
ServerName[3]: SERVER5
ServerName[4]:
SERVER1 is up!
SERVER3 is up!
SERVER2 is up!
SERVER5 is up!
```

You can see the format of the prompt, after at least 1 server name is specified, to then execute the code, you simply press enter on a blank parameter.

---

## Add Pipeline Support

Next up, lets add some pipeline support!  Let's use the scenario that you want to take the output of Get-ADComputer and then pass this on to our Get-ServerStatus.  For this to work correctly we need to amend the Parameter to accept values from the pipeline, we also need to keep the name of the parameter consistent with the output of Get-ADComputer.  Our amended Parameter will now look like this:

```powershell
Function Get-ServerStatus {
    [CmdletBinding()]
    Param(
        [Parameter(Mandatory = $true, ValueFromPipelineByPropertyName)]
        [string[]]$Name
    )
```

Which makes our code look like this:

```powershell
Function Get-ServerStatus {
    [CmdletBinding()]
    Param(
        [Parameter(Mandatory = $true, ValueFromPipelineByPropertyName)]
        [string[]]$Name
    )
    foreach ($Server in $Name) {
        if ((Test-NetConnection -ComputerName $Server -CommonTCPPort RDP).TcpTestSucceeded) {
            Write-Host "$Server is up!" -ForegroundColor Green
        }
        else {
            Write-Host "$Server is down!" -ForegroundColor Red
        }
    }
}
```

When we now run our script, lets try to pass a single result from Get-ADComputer to our function,  so entering the following PowerShell will return....

```powershell
PS C:\Users\JKelly> Get-ADComputer SERVER1 | Get-ServerStatus
SERVER1 is up!
```

Outstanding news! So, final test is to take multiple values from Get-ADComputer and pass this to our function;

```powershell
PS C:\Users\JKelly> Get-ADComputer -Filter * -SearchBase "OU=Domain Controllers,DC=LabDomain,DC=co,DC=uk" | Get-ServerStatus
SERVER3 is up!
```

Ah! - Problem.  Our Get-ADComputer cmdlet should return multiple servers.  Let's check how many servers we should expect by wrapping the Get-AdComputer cmdlet in brackets and adding ".Count" to it:

```powershell
PS C:\Users\JKelly> (Get-ADComputer -Filter * -SearchBase "OU=Domain Controllers,DC=LabDomain,DC=co,DC=uk").count
12
```

Looking at the results we know that our test should have ran against 12 servers.  The issue here is we do not have a "Process" statement in our function.  A process statement is essential if you want to pass multiple items over the pipeline and have all of these items processed.  Without this, the last item on the pipeline will be processed, everything else will be ignored.

So a blank function with a process statement would look like this:

```powershell
Function My-Function {
    [CmdletBinding()]
    Param(
        [Parameter(Mandatory = $true, ValueFromPipelineByPropertyName)]
        [string[]]$Name
    )
    Process{
        Write-host "all your code goes here"
    }
}
```

So now let's add a process statement to our function and re-test.  Adding the process statement should result in the code looking like this:

```powershell
Function Get-ServerStatus {
    [CmdletBinding()]
    Param(
        [Parameter(Mandatory = $true, ValueFromPipelineByPropertyName)]
        [string[]]$Name
    )
    Process {
        ForEach ($Server in $Name) {
            If ((Test-NetConnection -ComputerName $Server -CommonTCPPort RDP).TcpTestSucceeded) {
                Write-Host "$Server is up!" -ForegroundColor Green
            }
            Else {
                Write-Host "$Server is down!" -ForegroundColor Red
            }
        }
    }
}
```

Let's save and execute our code then use the Get-ADComputer cmdlet to test with multiple results once more:

```powershell
PS C:\Users\JKelly> Get-ADComputer -Filter * -SearchBase "OU=Domain Controllers,DC=LabDomain,DC=co,DC=uk" | Get-ServerStatus
SERVER6 is up!
SERVER2 is up!
SERVER7 is up!
SERVER5 is up!
SERVER9 is up!
SERVER10 is up!
SERVER11 is up!
SERVER12 is up!
SERVER13 is up!
SERVER1 is up!
SERVER14 is up!
SERVER3 is up!
```

Boom! So in the space of an hour we have a handy little tool that we can use or share!!

---

## Convert the Function to a Module

If you have followed this process so far, you will be thinking, every time I use PowerShell I'm going to have to run this script first, well yes and no.  We can leave it as is and you will need to execute the script first, or you can convert your script into a "Script Module".  This is a very simple process.

Let's assume that your script is called GetServerStatus.ps1.  Save a copy of your Script as GetServerStatus.psm1.

That is all that is required to create a module.  You will now want your module to be available automatically.  For this, a copy of the GetServerStatus.psm1 file needs to be placed within a sub-folder of the PowerShell modules folder.

The Powershell modules folder can be located by running the following PowerShell:

```powershell
$env:PSModulePath
```

This will return something similar to :

```powershell
PS C:\Users\JKelly\desktop> $env:PSModulePath
C:\Users\JKelly\Documents\WindowsPowerShell\Modules;C:\Program Files\WindowsPowerShell\Modules;C:\W
INDOWS\system32\WindowsPowerShell\v1.0\Modules;C:\Program Files\Microsoft System Center 2016\Virtua
l Machine Manager\bin\psModules\;C:\Program Files\Microsoft System Center 2012 R2\Operations Manage
r\Powershell\;C:\Program Files (x86)\NetApp\NetApp PowerShell Toolkit\Modules\.;C:\Program Files (x
86)\WindowsPowerShell\Modules\
```

That's horrific so let's make it a bit easier to read by splitting the values out:

```powershell
$env:PSModulePath -split ';'
```

This will output something a bit more readable:

```powershell
C:\Users\JKelly\Documents\WindowsPowerShell\Modules
C:\Program Files\WindowsPowerShell\Modules
C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules
C:\Program Files\Microsoft System Center 2016\Virtual Machine Manager\bin\psModules\
C:\Program Files\Microsoft System Center 2012 R2\Operations Manager\Powershell\
C:\Program Files (x86)\NetApp\NetApp PowerShell Toolkit\Modules\.
C:\Program Files (x86)\WindowsPowerShell\Modules\
```

We will want to select the user specific module area above.  In this case: C:\Users\JKelly\Documents\WindowsPowerShell\Modules and do the following:

1. Create a sub-folder called GetServerStatus
1. Copy your GetServerStatus.psm1 file to the previously created GetServerStatus Folder.

Now we are ready to check that the module is available.  To do this, simply enter the cmdlet below:

```powershell
 Get-Module -ListAvailable GetServerStatus
```

This should return output similar to this:

```powershell
    Directory: C:\Users\JKelly\Documents\WindowsPowerShell\Modules


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     0.0        GetServerStatus                     Get-ServerStatus
```

As the module is now in the default user Modules folder, it will be imported whenever you call it.  So all ready to use!

If you wanted to do the above steps via PowerShell:

```powershell
$PSPath = $env:PSModulePath -split ';'

$PSPath = ($PSPath[0]) + "\GetServerStatus"

If (Test-Path $PSPath) {
    Write-Warning "$PSPath Already exists"
}
Else {
    New-Item -ItemType Directory -Path $PSPath
    Copy-item c:\Scripts\GetServerStatus.psm1 -Destination $PSPath
}

Get-Module -ListAvailable GetServerStatus
```

---

## Summary and Next Steps

Hopefully you found this guide useful, I will create a follow up and cover some further tasks such as creating help information and making your module into a module manifest, publishing it to a repository and sharing with others!

if you want to go over any of this, give me a shout!
James
