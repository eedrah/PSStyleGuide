These documents are in an extremely rough state, not suitable for inclusion in the main guide yet.

<!-- MarkdownTOC depth=4 autolink=true bracket=round -->

- [More Formatting Rules?](#more-formatting-rules)
    - [Open braces on the same line](#open-braces-on-the-same-line)
    - [Closing braces always on their own line](#closing-braces-always-on-their-own-line)
    - [Prefer: param() begin, process, end](#prefer-param-begin-process-end)
- [Formatting](#formatting)
    - [Never use format cmdlets in functions and scripts](#never-use-format-cmdlets-in-functions-and-scripts)
    - [Ship Format Files When Outputting Custom Objects](#ship-format-files-when-outputting-custom-objects)
- [Using The .Net Framework](#using-the-net-framework)
    - [Use RequiredAssemblies rather than Add-Type](#use-requiredassemblies-rather-than-add-type)
    - [Use Add-Type rather than Reflection](#use-add-type-rather-than-reflection)
    - [Use Add-Type for small classes or PInvoke calls](#use-add-type-for-small-classes-or-pinvoke-calls)
    - [Prefer shipping binaries over large compilations](#prefer-shipping-binaries-over-large-compilations)
- [Comment Based Help](#comment-based-help)
    - [Always ship a about_ModuleName with modules](#always-ship-a-about_modulename-with-modules)
- [Performance](#performance)
    - [Prefer language features over cmdlets](#prefer-language-features-over-cmdlets)
    - [Know when to use .Net](#know-when-to-use-net)
- [Error Handling](#error-handling)
- [General Design Principles](#general-design-principles)
    - [Use custom objects](#use-custom-objects)
    - [Scripts vs Functions](#scripts-vs-functions)
    - [Always write CmdletBinding](#always-write-cmdletbinding)
    - [Prefer PipelineByPropertyName parameters.](#prefer-pipelinebypropertyname-parameters)
    - [Specify aliases for pipeline binding](#specify-aliases-for-pipeline-binding)
    - [Never forget the Common Parameters](#never-forget-the-common-parameters)
    - [Specify positional parameters, but don't use them](#specify-positional-parameters-but-dont-use-them)
    - [Specify short aliases, but don't use them](#specify-short-aliases-but-dont-use-them)
    - [Never specify Default values for mandatory parameters](#never-specify-default-values-for-mandatory-parameters)
    - [Always specify HelpText for mandatory parameters](#always-specify-helptext-for-mandatory-parameters)
    - [Use PsCmdlet.ThrowTerminatingError rather than throw](#use-pscmdletthrowterminatingerror-rather-than-throw)
    - [Use PsCmdlet.WriteError rather than Write-Error](#use-pscmdletwriteerror-rather-than-write-error)
    - [Use SupportsShouldProcess when appropriate](#use-supportsshouldprocess-when-appropriate)
    - [Modules ARE the Best Practice](#modules-are-the-best-practice)
    - [Use RequiredModules](#use-requiredmodules)
    - [Persisting Configuration](#persisting-configuration)
    - [Provide aliases in your modules](#provide-aliases-in-your-modules)
- [GOTCHAS](#gotchas)
    - [Beware string concatenation with +](#beware-string-concatenation-with-)
    - [Beware -match and -like](#beware--match-and--like)
    - [Beware -contains and -in](#beware--contains-and--in)

<!-- /MarkdownTOC -->

### More Formatting Rules?

#### Open braces on the same line
Code folding is nicer in many editors.

#### Closing braces always on their own line
Because that's how they're supposed to be!

#### Prefer: param() begin, process, end
That's the order PowerShell will execute it in

### Formatting

#### Never use format cmdlets in functions and scripts
#### Ship Format Files When Outputting Custom Objects

### Using The .Net Framework

#### Use RequiredAssemblies rather than Add-Type
#### Use Add-Type rather than Reflection
Avoid using `[System.Reflection]` to load assemblies when possible. Particularly avoid `LoadWithPartialName` (specify the full name instead).

#### Use Add-Type for small classes or PInvoke calls
TODO: Is this better than PowerShell Classes, for compatibility?

#### Prefer shipping binaries over large compilations
With PowerShell 5, security is tighter, and compiling code in memory will be frowned upon. Now that we have PowerShellGet and the PowerShell Gallery, there are few reasons 's no reason to avoid binaries.

TODO: Discuss: when does embedding C# code makes sense more sense than just compiling it every time?

### Comment Based Help

#### Always ship a about_ModuleName with modules

TODO
Discuss: Add a blank line between comment based help and function declaration?
Discuss: Minimum acceptable comment based help: Synopsis, Parameters, and an example for each parameter set (plus pipeline examples if you can contrive one)
Discuss: Benefits of MAML help files
Discuss: Other reasons to write about_topics

### Performance

#### Prefer language features over cmdlets
Prefer foreach(){} over ForEach-Object
Prefer .foreach and .where over cmdlets 
Prefer functions with process blocks over ForEach-Object

#### Know when to use .Net
Prefer writing wrapper functions to just calling .Net APIs
Discuss: System.IO file loading vs Get-Content (large files)
Discuss: Other examples of .Net API calls that are clearly faster?
Discuss: Casting for creating objects

### Error Handling

TODO
Discuss: Avoid depending on `$?` -- why?
Discuss: Never use `$Error` in scripts (always use -ErrorVariable)
Discuss: Interactively, always copy $Error[0] to $e or something


### General Design Principles

#### Use custom objects
Discuss: Add-Type vs PowerShell classes vs creating PSCustomObjects 
Discuss: Casting for creating objects
Discuss: PSTypeNames and formatting rules

#### Scripts vs Functions
Discuss: In PowerShell 3+, performance is such that there's very little penalty for using scripts.
Discuss: In PowerShell 3+, module autoloading means there's no discoverability penalty for writing functions (if you put them in a module) instead (except loading sets of them at a time).
Discuss: During development, always write scripts, which are automatically re-parsed if you edit the file.

#### Always write CmdletBinding

Let's just get this out of the way: all of your scripts should start life as this snippet:

```
[CmdletBinding()]param()
process{}
end{}
```

You can always ignore one of the blocks, and add parameters and such, but you should never write a script without CmdletBinding, and you should never write one without at least considering making it take pipeline input

#### Prefer PipelineByPropertyName parameters.
Discuss: This allows the most flexibility: piping objects and using scriptblocks to shape it for parameters. Unless you absolutely need to write a `begin` block and use this parameter in it, you probably should accept it on the pipeline.

Discuss: There is some testing overhead involved in that decision.

Discuss: You MUST do all your work in the process block if you do this.

#### Specify aliases for pipeline binding
You can use aliases to map parameters to property names of objects which might be piped to you while still keeping your parameter names clear and meaningful.

#### Never forget the Common Parameters
Particularly when splatting PSBoundParameters to the next function, if that function isn't `[CmdletBinding()]` you must remember to strip the common parameters if they're present.

#### Specify positional parameters, but don't use them
When writing at the command line, positional parameters are a blessing, but they can be confusing for future readers. You should always expose your parameters positionally when it makes sense, but you should rarely share a script that pases parameters positionally.

#### Specify short aliases, but don't use them
Again, for the sake of typing, it's particularly useful if you can provide two-letter aliases for each of your parameters such that every parameter has a two-letter or less name which is unique.

#### Never specify Default values for mandatory parameters
It doesn't do anything, and it confuses future readers.

#### Always specify HelpText for mandatory parameters
When prompted for a mandatory parameter, a user can request HelpText, but can't look at the documentation. It's frequently useful to duplicate at least the first sentence or two of the parameter help.

```
[Parameter(Position=1, Mandatory=$true, ValueFromPipeline=$true,
ValueFromPipelineByPropertyName=$true, HelpText='The name of the file to read')]
[Alias('PSPath','FullName','Path')]
[String]$File
```


#### Use PsCmdlet.ThrowTerminatingError rather than throw
#### Use PsCmdlet.WriteError rather than Write-Error
Discuss: These need example output to explain why they're better
Discuss: a common template (from the Microsoft team) for throwing errors

```
# Utility to throw an errorrecord
function ThrowError
{
    param
    (        
        [parameter(Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [System.Management.Automation.PSCmdlet]
        $CallerPSCmdlet,

        [parameter(Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [System.String]        
        $ExceptionName,

        [parameter(Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [System.String]
        $ExceptionMessage,
        
        [System.Object]
        $ExceptionObject,
        
        [parameter(Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [System.String]
        $ErrorId,

        [parameter(Mandatory = $true)]
        [ValidateNotNull()]
        [System.Management.Automation.ErrorCategory]
        $ErrorCategory
    )
        
    $exception = New-Object $ExceptionName $ExceptionMessage;
    $errorRecord = New-Object System.Management.Automation.ErrorRecord $exception, $ErrorId, $ErrorCategory, $ExceptionObject    
    $CallerPSCmdlet.ThrowTerminatingError($errorRecord)
}
```


#### Use SupportsShouldProcess when appropriate
Discuss: when is this critical (-whatif) and optional (-confirm_
Discuss: when should you call PSCmdlet.ShouldProcess vs PSCmdlet.ShouldContinue (-Force)

```
[CmdletBinding(SupportsShouldProcess=$true, ConfirmImpact="Medium")]
param([Switch]$Force)

$RejectAll = $false;
$ConfirmAll = $false;

foreach($file in ls) {

   if($PSCmdlet.ShouldProcess( "Removed the file '$($file.Name)'",
                               "Remove the file '$($file.Name)'?",
                               "Removing Files" )) {

      if($Force -Or $PSCmdlet.ShouldContinue("Are you REALLY sure you want to remove '$($file.Name)'?", "Removing '$($file.Name)'", [ref]$ConfirmAll, [ref]$RejectAll)) {
         "Removing $File"


      }
   }
}
```

#### Modules ARE the Best Practice
Discuss: Scope isolation, hiding helper functions, sharing state between functions
Discuss: Organizing related functions
Discuss: Discoverability, Packaging, Sharing
Discuss: Dependency Management, Versioning
Discuss: Internationalization

#### Use RequiredModules
The problems with this have gone away with autoloading, and this is the only way to let PowerShellGet manage dependencies for you.

#### Persisting Configuration
My choice: Configuration module. Otherwise, use clixml (or XAML) to persist to AppData (TEST: you shouldn't store configuration in your module folder, as it may not survive upgrades (in PowerShell 3 & 4 there was no side-by-side loading))


#### Provide aliases in your modules
You should feel free to create and use aliases within your modules. In some cases, you can even improve readability by using an alias without the verb, or shortening command names.

For exported aliases, follow the guidance of Microsoft ("ip" for import, "s" for set, "g" for get, "r" for remove, etc.), make up somethign for your nouns.

Use `New-Alias ... -ErrorAction SilentlyContinue` to avoid overwriting existing aliases.  




### GOTCHAS

#### Beware string concatenation with + 
You should always wrap this with parentheses, because otherwise it can break (e.g. when passing a string as a parameter.

#### Beware -match and -like
They quietly cast objects to strings (or arrays of strings)

#### Beware -contains and -in
They work on ARRAYS not strings


   Add-TextSlide "Use Language Features" @(     # Choose Language over Cmdlets
      ,"DO NOT pass collections to ForEach-Object"
      ,"INSTEAD use the language foreach(){}"
      ,"EXCEPTIONS:"
      ,"`tYou're generating output"
      ,"`tYou're modifying the collection"
      ,"RATIONALE:"
      ,"`tNamed variables are easier"           # Easier to edit + harder to forget
      ,"`tIt's much faster"                     # Up to 2x faster
   )
   Add-TextSlide "Understand .Net"        @(    # You should understand the .Net underpinings
      ,"AVOID appending to arrays in a loop"
      ,"INSTEAD assign output from the loop"
      ,"EXCEPTIONS:"
      ,"`tAppending to multiple collections"
      ,"`tUsing Lists instead of arrays"
      ,"RATIONALE:"
      ,"`tCopying is slow"
      ,"`tPipeline output uses ArrayList"
      ,"`nEasier to read and understand"
   )
   Add-TextSlide "Understand .Net"        @(    # You should understand the .Net underpinings
      ,"AVOID appending to strings"
      ,"INSTEAD assign output from the loop"    # Use $OFS
      ,"EXCEPTIONS:"
      ,"`tWhen it's really short"
      ,"`tWhen it's a short loop"
      ,"RATIONALE:"
      ,"`tCopying is slow!"                     # Remember:
      ,"`tJoin strings with $OFS"
      ,"`nEasier to read and understand"
   )
   Add-TextSlide "Single Output Types" @(       # Comprehension
      ,"AVOID mixing output types"
      ,"INSTEAD Write-Host or Write-Warning"    # I've been seeing code using 'echo' to print text and THEN having the output.
      ,"You can also use -Verbose or -Quiet"    # Use Write-Verbose, since you're CmdletBinding, that way you can turn it off!
      ,"RATIONALE:"
      ,"Pipelines and assignments"              # Obviously you shouldn't if you echo or output text along with your objects
      ,"This should go without saying"
   )
   Add-TextSlide "Strongly Type Parameters" @(  # Be strongly typed on your inputs when you should be.
      ,"Prefer strongly typed inputs"           # Stops errors before they start (literally)
      ,"Avoid String with ParameterSets"        # Avoid String because PowerShell will cast anything to it.
      ,"Always be checking"                     # Strong Types on parameters keep you safe :)
      ,"When Passing on Parameters"             # If you're accepting a parameter which you will pass directly to another command, you should be at least as strongly typed as the command you're calling. If you're going to Test-Collection -ComputerName $InputObject ... make sure that your Param block says: [String[]]$InputObject ...
      ,"EXCEPTIONS:"
      ,"`tMultiple types are acceptable"        # When a parameter could be passed as either of two different types and you don't want to overload the function with a different parameter NAME to create a new parameterset.
      ,"RATIONALE:"
      ,"`tUsers don't have to guess"            # This stuff shows up in the man pages
      ,"`tScripts will fail early!"             # Better early than often
   )
   Add-TextSlide "Output Formatting" @(
      ,"Don't format function output. Ever"
      ,"INSTEAD:"
      ,"`tOffer a -Formatted parameter"
      ,"`tInclude formats.ps1xml in module"
      ,"`tPSTypeNames works!"
   )
   Add-TextSlide "Output Formatting 2" @(
      , "Only output one kind of thing"         # PowerShell buffers output and results are weird otherwise
      , "Otherwise, Name Appropriately"         # When you have a really good reason, make it clear in your function name that you're output is mixed
      , "RATIONALE:"             
      , '&{ $MyInvocation | gm' +               # The output of this (and many variations) will convince you of the buffering
        'write-host "----------"' +
        '$MyInvocation | gm | fl }'
      , '&{ ls | gm' +                          # The output of this (and any variations) will convince you not to mix output types
        'write-host "----------"' +
        'ls }'
   )
   

   Add-TextSlide "Don't Duplicate Work"         # Don't sort things that are already sorted
   Add-TextSlide "Logging from Scripts"         # Logging from scripts (best and worst ways to do it)
   Add-TextSlide "Code-Signing"                 # Code-signing, security, and multiple developers
   Set-Slide 0
