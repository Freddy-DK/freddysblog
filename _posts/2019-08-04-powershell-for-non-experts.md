---
layout: post
title: "PowerShell for non-experts…"
date: 2019-08-04 05:57:03
categories: ["Docker", "NavContainerHelper", "PowerShell"]
tags: ["Docker", "NavContainerHelper", "PowerShell"]
permalink: /2019/08/04/powershell-for-non-experts/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

With the release of NAV and Business Central images on Docker, a lot of people who are not familiar with PowerShell will be using PowerShell. They will be handed a script to run and they will run it. Then something doesn’t work as expected – and somebody (might be me) will tell them to add a parameter called xyz. That might seem simple for people who use PowerShell on a daily basis, but not if you have never written PowerShell before…

This blog post will take you through some of the basic elements of PowerShell. Enough to successfully use NAV and Business Central containers on Docker at least (I hope).

**Note:** _This isn’t a PowerShell tutorial as such and there might be terminologies I have wrong or things I have misunderstood, but if you can follow this – you should be able to use PowerShell to start NAV or Business Central Containers on Docker using PowerShell and NavContainerHelper_

# What is PowerShell?

PowerShell is an extremely powerful scripting language. Every line you enter is passed through an interpreter and then executed. You can define functions and variables and write code just like in any other programming language. You have full support for .net and one of the very very nice thing about working with PowerShell is, that you can mark any arbitrary line and execute them individually in PowerShell ISE or VS Code. A script doesn’t have to be compiled and run sequentially.

PowerShell exists in a 32 bit version and a 64 bit version. The NavContainerHelper module only supports the 64 bit version.

# Variables

A variable is a name preceded by a dollar-sign. You declare the variable by assigning the variable to a value.

$s = 'Hello World'

Assigns the string Hello World to the variable $s.

$n = 45

Assigns the integer number 45 to $n.

$a = @( $false, 'one', 2, 3.1415)

Assigns an array consisting of a boolean, a string, an integer and a decimal number to $a. Yes, you can have different types of values in arrays, PowerShell doesn’t care.

PowerShell is very relaxed in type checking and if you want to test whether $s has a value, you can just write:

if ($s) { Write-Host '$s has a value' }

If $s is $null or en empty string, it will not print the string.

The same with numbers

if ($n) { Write-Host '$n is not 0' }

and the same works with arrays as well. It will be true if the array contains anything, but note that an array of empty values is still false.

# Comparison operators

Unlike most other programming languages, comparison operators in PowerShell consists of a hyphen followed by a few characters describing the operator:

-   \-eq = equals
-   \-ne = not equals
-   \-gt = greater than
-   \-ge = greater or equal
-   \-lt = less than
-   \-le = less or equal
-   \-like = like
-   \-notlike = not like
-   and more…

Note, that all string comparisons are case insensitive. If you want case sensitive comparisons, you need to insert a ‘c’ after the hyphen (-ceq = case sensitive equals)

Note that there is NEVER a space after the hyphen.

# The difference between ‘ and “

String literals can be specified in two ways in PowerShell:

$s1 = 'Hello World'
$s2 = "Hello World"

Both variables will contain a string called Hello World, the constructs seems similar, but there is one very big difference.

String literals with double quotes (“) will be interpreted by PowerShell and variables inside unpacked.

$s = 'World'
$s1 = 'Hello $s'
$s2 = "Hello $s"

$s1 will now contain the value **Hello $s** and $s2 will be **Hello World**.

You can even insert entire expressions in the string by inserting it inside $() like:

$s = 'World'
$s2 = "Hello $($s.ToUpperInvariant())CHAMPION"

$s2 will become **Hello WORLDCHAMPION**. Another construct you might see is:

$s = 'World'
$s1 = "Hello ${s}champion"
$s2 = "Hello $($s)champion"

Here both $s1 and $s2 will become **Hello Worldchampion**. I have stopped using the first construct (primarily because you cannot search for $s and usages of the variable), but you might still see this occasionally.

Another special construct for string literals you might see are here-strings.

$s = @"
This is line1
This is line2
"@

You can also use single-quotes for here-strings and the same rules apply, where here-string with double-quotes will have the content interpreted by PowerShell.

Quick side-note, the above string will contain one line-break (the one between the lines). The line-break after the here-string start (@”) and the line-break before the here-string end (“@) is not included.

You can use any character inside here-strings. The here-string won’t end until “@ on a new line. You might also see multi-line string literals (remove the two @’s), in that case, the line-break in the string is just treated as part of the string and in the above sample, your string would have 3 line-breaks.

# The hyphen (-)

The hyphen is used as a minus operator when subtracting numbers in expressions or it  is used to precede a comparison operator or a parameter for a function.

$n = 45 - 20

$n will become 25.

$v = $s -like "g\*"

$v will become $true if $s starts with a g or G.

New-BcContainer -accept\_eula -containerName "test"

Calls the function New-BcContainer with 2 parameters (accept\_eula and containerName). See more about parameters later.

Note that there ONLY a space after the hyphen if the hyphen is used as a minus.

# The grave (\`)

The grave is used as an escape character in string literals – or it is used to indicate that the current line continues on the next line.

$s = "This is line1\`nThis is line2"

$s will become exactly the same string as the last sample in the string literals examples. A string, which contains a newline character between This is line1 and This is line2.

The grave symbol also escapes quotation marks

$s = "String with \`"quotation marks\`""

$s will become **String with “quotation marks”**. The escape characters can also be used for tabs and other things, but those are seldom seen.

The grave symbol’s most important meaning is probably to indicate that a line continues on the next line. This command

New-BcContainer -accept\_eula \`
                -containerName "test"

is exactly the same as

New-BcContainer -accept\_eula -containerName "test"

Note that you cannot place anything after the grave symbol. Even a single space will cause the grave symbol to lose it’s meaning.

Also remember, if you use the grave to split parameters, you cannot out-comment a line like this:

New-BcContainer -accept\_eula \`
                -containerName "test" \`
#                -accept\_outdated \`
                -imageName "mcr.microsoft.com/businesscentral/onprem"

The above will run New-BcContainer with Accept\_eula and the containername and then it will try to execute -imageName as a command, giving you an error. You need to remove the parameter line you don’t want.

A common mistake is either to forget the grave symbol – or forget to remove it. This

$str = "This is a test"
Write-Host -foreground Blue "test" \`
Write-Host $str

Will print out the string test Write-Host This is a test in blue, where the intention probably was this

$str = "This is a test"
Write-Host -foreground Blue "test"
Write-Host $str

printing out test in blue and on the following line **This is a test**.

Another common mistake is to have spaces after the grave symbol. You will be able to detect this at the syntax highlighting:

![spaceaftergrave](/assets/images/2019/powershell-for-non-experts/spaceaftergrave.png)

In the first Write-Host, the -Object Parameter has the same color as the -Object parameter in line 3, but in #2, the -Object name has the same color as a command. A space after the grave symbol means that the grave symbol no longer works as a grave symbol, but now just become part of the line. The result of this would be:

Test #1

-Object : The term '-Object' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the name, or if a path was included, verify 
that the path is correct and try again.
At line:3 char:5
+ -Object "Test #2"
+ ~~~~~~~
+ CategoryInfo : ObjectNotFound: (-Object:String) \[\], CommandNotFoundException
+ FullyQualifiedErrorId : CommandNotFoundException

Test #3

(It sees -Object in line 2 as a command and it is unknown)

Where you really wanted this:

Write-Host \`
-Object "Test #1"
Write-Host \`
-Object "Test #2"
Write-Host -Object "Test #3"

Test #1
Test #2
Test #3

# Functions/Cmdlets

I won’t go into detail about function definitions in this post, just mention, that the way you define the simplest function of them all is like this:

function Test {
    Write-Host "Hello World"
}

and with a parameter (which has a default value) like this

function Test(\[string\] $name = "World") {
    Write-Host "Hello $name"
}

Calling these functions are done using:

Test

which will print **Hello World** on the host, or

Test -name "Freddy"

which will print **Hello Freddy** on the host. You can also omit the parameter description and use

Test "Kristiansen"

which will write **Hello Kristiansen**. All parameters you transfer without a parameter description are transferred to parameters in the function after the order defined.

There is much more to function definitions, but that is for another time.

# Switch vs. Boolean

A special parameter type I want to touch upon is the switch. Have a look at these functions

function test1(\[switch\] $accept\_eula, $name) {
    if ($accept\_eula) {
        Write-Host "$name accepted the eula"
    }
}

function test2(\[boolean\] $accept\_eula, $name) {
    if ($accept\_eula) {
        Write-Host "$name accepted the eula"
    }
}

They seem very similar, but the way you call them are very different.

Test1 -accept\_eula "Freddy"
Test2 -accept\_eula $true "Freddy"

The boolean parameter is a “normal” parameter, where you have to specify the value after the parameter description. You could omit the parameter description and write Test2 $true – but that doesn’t say much.

Both calls will result in **Freddy accepted the eula**.

If you try to add $true to the Test1 call, the sample will now say **True accepted the eula**, because $true was just transferred into the next parameter and the last parameter value (“Freddy”) is now just ignored. If you have a boolean variable and you need to transfer that to accept\_eula function above you need to call the function like this:

$accept = $true
Test1 -accept\_eula:$accept "Freddy"
Test2 -accept\_eula $accept "Freddy"

In NavContainerHelper, I like using switches because of the readability.

# Host vs. Output

If we replace the function from the functions section with:

function Test(\[string\] $name = "World") {
    Write-Output "Hello $name"
}

and re-run the function, it looks like nothing changed, but in fact it is a HUGE change.

Objects sent to the host stream are printed on the host. In Write-Host you can define colors and control how things are printed.

Objects sent to the output stream are return values from the function. If you change the way you call the function and instead use:

$s = Test -name "Freddy"

Then $s will have the value **Hello Freddy** if the function is using Write-Output and $s will be $s will be **$null** if the function is using Write-Host.

There are three ways of returning values from a function. If you change the function definition to

function Test {
    Write-Output "First value"
    "Second value"
    return "Third value"
    Write-Output "Forth value"
}

and call the function using **Test**, you will get

First value
Second value
Third value

Write-Output will add the value to the output stream. A line with an expression will also just place this expression on the output stream, the same happens if you call a function which returns a value. return will also place the value on the output stream, but it will also terminate function execution (hence no fourth value).

When all execution of a function/cmdlet in PowerShell is done, the output stream is written on the console.

Note that any line executed that returns a value will be added to the output stream. When you write $s to get the value of $s, PowerShell will add that to the output stream and then because execution stop – it will be written to the console. If you write 1..10<enter> 10 numbers will be added to the output stream.

# The pipe (|)

As we have seen, functions, statements or expressions can return an object or an array of objects and this is done by writing the objects to the output stream. Adding a pipe will cause the objects in the output stream to be sent to the cmdlet after the pipe.

You can f.ex. pipe our function from the previous section to Write-Host and use Red color

Test | Write-Host -ForegroundColor red

will print

First Value
Second Value
Third Value

You can also pipe the objects through a where clause:

test | Where-Object { $\_ -like "second\*" } | Write-Host -ForegroundColor red

or sort them descending. Sorting can be done on any property in the object or on the value. In this case the objects are sorted alphabetically descending.

test | Sort-Object -Descending | Write-Host

And you can pipe the objects in the output stream through any number of pipes.

In NavContainerHelper I haven’t been very good at implementing pipes.

# The percent (%)

The percent character is really just an alias for the CmdLet called **ForEach-Object** and means that for every object, coming through the pipeline, the following scriptblock is executed and the object is available as $\_ inside the scriptblock.

Get-ChildItem "c:\\windows\\system32\\\*.exe" | % {
    Write-Host $\_.Name
}

Will write the name of all .exe files in the c:\\windows\\system32 folder on the host.

# Installing and updating modules

Installing the latest version of NavContainerHelper is done using this command:

Install-Module navcontainerhelper -force

After this, you will find the module in **Get-InstalledModule**, which as the name indicates lists the modules you have installed.

When you have use the module the first time (or imported the module using **Import-Module**) you will also find the module in **Get-Module**, which will list the modules, which are currently loaded.

Updating the NavContainerHelper to the latest version is done using this command:

Update-Module navcontainerhelper -force

If the module is loaded, you will have to restart PowerShell to load the new version.

Note that old versions of the module are not uninstalled. Listing all installed versions is done using:

Get-InstalledModule -Name navcontainerhelper -AllVersions

and if you like, you can uninstall all installed versions and install the newest instead of updating the module using:

Get-InstalledModule -Name navcontainerhelper -AllVersions | Uninstall-Module

If the module is loaded (visible in Get-Module), then it will stay in memory until you restart PowerShell or remove the navcontainerhelper module from memory using:

Remove-Module navcontainerhelper

If you for some reason get stuck with more than one loaded version in memory (intellisense shows multiple functions with the same name) – uninstall all and install the latest. NavContainerHelper doesn’t clean up directories or anything when uninstalling.

# Conclusion

I hope that this covers most things people need in order to use the NavContainerHelper and PowerShell to create and work with NAV and Business Central containers.

If you encounter things, which you think should have been described here, let me know in the comments and I will try to include them if I think it is appropriate.

I hope this is helpful

**Freddy Kristiansen**  
_Technical Evangelist_
