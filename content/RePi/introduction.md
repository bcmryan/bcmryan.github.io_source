+++
title = "Introduction"
weight = "10"
+++

The Reproducible Raspberry Pi (RePi) server build project aims to walk you step by step through the stages of building a Raspberry Pi server that is stable, secure and, as the name suggests, reproducible.
The specific setup that will be documented in this tutorial consists of a Raspberry Pi 4 connected to a hard drive array which will serve files directly to devices on the local network and interface with users via other means (specifically through a containerized media player).
At the completion of this project you should have learned some basic Linux commands and have an active understanding regarding the functioning of their server's components.

Reproducibility is of the utmost importance to this project.
The hope is that following the completion of this project were the worst to happen -- be it a hard drive failure, the Pi overheating to the point of no longer functioning, etc. -- you would have the skills to quickly rebuild their server with the least amount of friction and/or data loss.
To achieve this goal, this project relies solely upon easily accessible consumer grade equipment and open-source computer programs, most of which are operated solely via the command line and/or through configuration (config) files.
While this might seem a bit daunting to some, once you learn how to manage your server via the command line, you will be amazed at how easy it is to back up and restore your settings through the power of the written word.
Following this approach, you will no longer need to worry about if they checked the right box in the settings bar, because if you properly backed up your scripts and files, you should be able restore your server to working order by selecting and committing the right text document.

I have two main motivations for creating this project.
First, I too often have found myself copying and pasting lines from tutorials online into my terminal window and watching them return the correct result without actually learning how or why the commands work.
Such a tutorial is not what I intend to create here.
This project should walk you through the process of understanding why certain commands might work better in certain scenarios and why others might be more applicable elswhere.
Second, and on a very related note, I would also like to document the rebuilding of my Raspberry Pi server from the ground up.
I would like to complete my build in the fewest steps possible and allow it to be resilient enough to remain reliable no matter what I throw at it.
And if I do in fact throw too much at it, I would like to have clear documentation about how to get it back to a working state.
My personal goal is to have my server be a nearly disposable appliance (at least regarding software) rather than something held together by love and duct tape that only I can operate.
Regarding the writing of this tutorial, I believe that if I cannot properly explain my motivations for certain choices that I make or if I cannot explain in a step-by-step manner how to complete a task, I must be doing something wrong.
This project should help me document how I created the server that stores the files I depend on every day and should allow you to do the same.

I hope this has piqued the interests of those like me who do not want to worry about breaking their server after installing a new application or running an update.
If so, join me in my journey of creating a Reproducible Raspberry Pi.

## Unix philosophy
This project is based around the rather simple yet powerful maxim known as the [Unix philosophy](https://en.wikipedia.org/wiki/Unix_philosophy).
This states that each component in a system should "do one thing and do it well" while working with other similarly small components to build a bigger structure that produces the end result that you desire.
While this is usually thought of in the realm of software -- one small program does a single calculation and then sends its result to another program which performs another calculation and so on -- the idea of (replaceable) modularity can also be applied to hardware as well: for this build, if any single component -- a cable, a hard drive, the hard drive enclosure and, yes, even the Pi itself -- were to fail, it could be replaced without the entire system failing to the point of not being able to be rebuilt.
Thus, each component has a simple and well-documented job; if it cannot perform that task, it should be and can be replaced.

## Infrastructure as code
In a similar vein, the idea of [infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_code) comes into play.
The idea behind this structure is that information technology (IT) professionals should treat the tools (technological infrastructure) that they use the same way that they treat code by standardizing their hardware and software, iterating (while making sure that they are keeping a record of their changes, a process known as version controlling) and automating tasks instead of performing them manually whenever possible.
The goal of these processes is to produce a more consistent and accurate end result and avoid the idiosyncrasies that individuals may introduce into a system's workflow.
(Mistakes are what make us human.)

## Command line interface
In order to methodically build our server up from a collection of modular and replaceable components (Unix philosophy) and systematically automate its tasks (infrastucture as code), we must be able to interact with our its operating system.
This tutorial nearly exclusively eludes to physically typing commands into a terminal emulator via the [command line interface](https://en.wikipedia.org/wiki/Command-line_interface) (CLI).
There are two main benefits of interacting with a computer via the command line in comparison to a graphical user interface (GUI) that are immediately apparent: (i) it is a more exact way of telling your operating system or an application what you want it to do (anyone who has ever dealt with the pain of trying to specify how big a partition should be using macOS's Disk Utility's pie chart can testify to that), and (ii) it is much more reproducible -- you can easily copy and paste a command that you wrote a year later into a terminal emulator and get the same result (assuming the underlying commands have not changed), while nagivating through pages and pages of settings can really be an exercise in frustration.
I want to be clear about one thing though: using the CLI does take some practice, and there are certain tasks -- especially ones that you do not need to repeat -- that may be more easily done with a GUI.
For instance, you can launch a new Firefox window via the command line, but that is something that I have personally never done, nor can I think of a good reason to ever do.
Use the right tool for the job.

That being said, when trying to steamline the build of a server made of consumer grade parts, I ([and others](https://medium.com/linuxforeveryone/the-real-reason-linux-users-love-the-command-line-e8043f583028)) truly believe that the command line and config files are absolutely the best tools for the job.
You cannot copy and paste button presses in a settings menu, but you absolutely can save commands you have entered into a text file to be used at a later date.

