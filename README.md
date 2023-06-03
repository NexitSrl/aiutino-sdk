# AIutino-SDK

Software Development Kit for AIutino System

This is the repository of the AIutino SDK. 
The SDK is provided for various languages and allows easy integration of cusomers solution with the AIutino System

## AIutino SDK for Java

The AIutino SDK is build under Java 8 and is distributed in .jar format to be easy to import into any IDE project.
Currently it requires **Java 8** or higher version.

## AIutino SDK for .NET

The AIutino SDK for .NET is build in C# and can be used in any .NET environment

Current version is released for **netstandard 2.0**, **.net 4.8**, **core net 6.0**.

You can download single dll or use the nuget package. To use the offline nuget package do the following:

1 download it to a local folder (say: *c:\\temp\\nugetRepo\\*)
2 add a **nuget.config** file in your solution folder containing the following text (put the previous folder name in *value*)

    <configuration>
        <packageSources>    
            <add key="github-local" value="c:\temp\nugetRepo\" />    
        </packageSources>
    </configuration>

3 add the package in Nuget command line (set *--version* with desired version)

    dotnet add package com.nexitsrl.aiutino.apiclient --version 1.0.0

## AIutino SDK for Python

*Coming soon*

