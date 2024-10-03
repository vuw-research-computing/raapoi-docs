# Basic Commands
### The _vuw_ Commands

In an effort to make using Rāpoi just a bit easier, CAD staff have created commands to help you view useful information.  We call these the _vuw_ commands.  This is because all the commands begin with the string _vuw_.  This makes it easier to see the commands available to you.  If, at a command prompt you type _vuw_ followed immediately by two _TAB_ keys you will see a list of available commands beginning with _vuw_.  Go ahead and type vuw-TAB-TAB to see for yourself.

The commands available as of this update are:

| Command      | Description                          |
| :---------- | :----------------------------------- |
| _`vuw-help`_    |            Prints this help information  |
| _`vuw-job-report`_    |      Provides some summary information about a job  |
| _`vuw-quota`_    |           Prints current storage quota and usage  |
| _`vuw-partitions`_    |      Prints a list of available partitions and the availability of compute nodes  |
| _`vuw-alljobs`_    |         Prints a list of all user jobs  |
| _`vuw-myjobs`_    |          Prints a list of your running or pending jobs  |
| _`vuw-job-history`_    |     Show jobs finished in last 5 days  |
| _`vuw-job-eff`_    |         Show efficiency of your jobs. Use vuw-job-eff --help for more information  |

!!! tip
    If you are unable to use these commands (e.g. with an error message "command not found") then double check you have the "config" module loaded (i.e. enter the command `module load config`).

### Linux Commands

Rāpoi is built using the Linux operating system. Access is primarily via command line interface (CLI) as opposed to the graphical user interfaces (GUI) that you are more familiar with (such as those on Windows or Mac) Below are a list of common commands for viewing and managing files and directories (replace the file and directory names with ones you own):

**ls** - This command lists the contents of the current directory

* _`ls -l`_ This is the same command with a flag (-l) which lists the contents with more information, including access permissions
* _`ls -a`_ Same ls command but this time the -a flag which will also list hidden files. Hidden files start with a . (period)
* _`ls -la`_ Stringing flags together

**cd** - This will change your location to a different directory (folder)

* _`cd projects/calctest_proj`_
* Typing _`cd`_ with no arguments will take you back to your home directory

**mv** - This will move or rename a file

* _`mv project1.txt project2.txt`_
* _`mv project2.txt projects/calctest_proj/`_

**cp** - This allows you to copy file/s and/or directories to defined locations. The ```cp``` command works very much like ```mv```, except it copies a file instead of moving it. 
The general form of the command is **cp _source destination_**, for example:

* ```cp myfile.txt myfilecopy.txt```

Further examples and options can be seen [here](https://www.howtoforge.com/linux-cp-command/).

**rm** - This will delete a file

* _`rm projects/calctest_proj/projects2.txt`_
* _`rm -r projects/calctest_proj/code`_
The _`-r`_ flag recursively removes files and directories

**mkdir** - This will create a new directory

* _`mkdir /nfs/home/myusername/financial`_

To find more detailed information about any command you can use the manpages,
eg:  _`man ls`_

### Learning the Linux Shell

A good tutorial for using linux can be found here:
[Learning the linux shell](http://linuxcommand.org/lc3_learning_the_shell.php).

[Software Carpentry](http://swcarpentry.github.io/shell-novice/) also provides a good introduction to the shell, including [how to work with files and directories](http://swcarpentry.github.io/shell-novice/03-create/index.html).
