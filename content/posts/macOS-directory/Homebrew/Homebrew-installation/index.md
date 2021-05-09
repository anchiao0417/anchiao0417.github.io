---
title: "How to solve Homebrew installation failure?"
date: 2021-04-07T08:06:25+06:00
description: "xcode-select: error: invalid developer directory /Library/Developer/CommandLineTools"
menu:
  sidebar:
    name: Homebrew Installation Troubleshooting
    identifier: macOS-directory-Homebrew-installation
    parent : macOS-directory-Homebrew
    weight: 10
hero: images/homebrew.jpg
---

If you got this error message while installing Homebrew on macOS, follow my steps for troubleshooting!

```shell
    The operation couldn’t be completed. (NSURLErrorDomain error -1012.)
    ==> Installing the Command Line Tools (expect a GUI popup):
    ==> /usr/bin/sudo /usr/bin/xcode-select --install
    xcode-select: note: install requested for command line developer tools
    Press any key when the installation has completed.
    ==> /usr/bin/sudo /usr/bin/xcode-select --switch /Library/Developer/CommandLineTools
    xcode-select: error: invalid developer directory '/Library/Developer/CommandLineTools'
    Failed during: /usr/bin/sudo /usr/bin/xcode-select --switch /Library/Developer/CommandLineTools
```

## Download CommandLine Tools

Command Line Tools is the basic requirements for downloading homebrew, if you haven't got one, simply download it  manually on [Apple Developer](https://developer.apple.com)

If you're able to log in and download the latest version of it, you should be good to go.

However, in my case, I get this message whenever I try to log in to the download page:

{{< alert type="warning" >}}
We are unable to process your request. An unknown error occurred..
{{< /alert >}}

It's hard to get a clue of what makes it happen since it says 'unknown error'. I contacted Apple Customer service for help, unfortunately they couldn't address the problem. Apple should fix this issue seriously.

## Download Xcode from Appstore

After some time of research, I found that CommandLineTools is installed if downloading the App 'Xcode' from Appstore, just note that in this way, the CommandLineTools might be in other directory, not as it should be in the homebrew official install script.

So here we have the solution, copy the whole homebrew install script , then simply replace

**/Library/Developer/CommandLineTools**

to

**/Applications/Xcode.app/Contents/Developer**

You can also get this install file here in my [Github](https://github.com/anchiao0417/homebrew/blob/main/install.sh)

Download the script file, open up your terminal and cd to where the file is, run it as the command:

```shell
    ./install.sh
```

That's it! The rest of the download process should be continued.

I believe in newer macOS version, XCode has changed its installed path. While Homebrew install script still remains the old path which causes the problem.

I hope you find this article informative and helpful, If you have any questions or comments, please feel free to leave them below.