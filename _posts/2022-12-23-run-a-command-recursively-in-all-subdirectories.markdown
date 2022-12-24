---
layout: post
title:  "Run a command recursively in all subdirectories"
author: Arun
tags: [programming, python, linux]
---

Suppose you wish to run a linux command in the current working directory, and in all of its subdirectories recursively. For example, I discovered that sometimes, running 'make modules_install' compresses all the modules as a .xz file in the various subdirectories. I read that the kernel does not care if a module is compressed, it can load the module in any case, but I did not know that at the time. I initially considered running "xz --decompress " on every .xz file, but there are simply too many subsystem and driver directories, and it would extremely tedious. So I wrote a little script to do it:

[run_cmd.py](https://gist.github.com/arunvijayshankar/a61fb0cd840f86e5f9ad4d4132497bf9)

```

"""
* Script to run a system command recursively in every subdirectory 
* in a directory, including the current working directory
* Usage: run_cmd.py -[si] -c <command-to-run>
*
* Options:
*   -i 
*       Interactive mode. Prompts user to choose whether or not to 
*       run the command in a directory
*   -s
*       Skips the command for the working directory
*   -c
*       Command to be run
*
* Example: $python run_cmd.py -s -c <command-to-run>
* This will run 'ls -l > contents.txt' in every subdirectory in the tree
* recursively, but will not run it in the working directory
*
* Author: Arun Vijayshankar
"""

import getopt
import os
import sys

def recursive_run_cmd_inter(root, cmd):
    for file in os.listdir(root):
        d = os.path.join(root, file)
        if os.path.isdir(d):
            os.chdir(d)
            run = input("Do you want to run " + cmd + " in: \n" + os.getcwd() + " [Y/No]")
            if str(run).upper() in ['Y', 'YES']:
                print("running " + cmd + " in: " + os.getcwd())
                os.system(cmd)
            recursive_run_cmd_inter(d, cmd)

def recursive_run_cmd(root, cmd):
    for file in os.listdir(root):
        d = os.path.join(root, file)
        if os.path.isdir(d):
            os.chdir(d)
            print("running " + cmd + " in: " + os.getcwd())
            os.system(cmd)
            recursive_run_cmd(d, cmd)
            
def entry():
    argv = sys.argv[1:]
    try:
        options, args = getopt.getopt(argv, "s:i:c", [])
    except:
        print("Usage: run_cmd.py -[si] -c '<command-to-be-run>'")

    root = os.getcwd()
    cmd = args[0]
    print(cmd)

    if '-i' in options[0]:
        recursive_run_cmd_inter(root, cmd)
    if '-s' in options[0]:
        if '-i' in options[0]:
            recursive_run_cmd_inter(root, cmd)
        else:
            recursive_run_cmd(root, cmd)
    else:
        os.chdir(root)
        print("running " + cmd + " in: " + os.getcwd())
        os.system(cmd)
        recursive_run_cmd(root, cmd)

if __name__ == "__main__":
    entry()

```

I tested it out on Ubuntu 20.04, and it seems to work. An addition that could be made is maybe the script can take a list of directory in which to run the command and run it in those only. I mightget around to doing this someday very soon. 
