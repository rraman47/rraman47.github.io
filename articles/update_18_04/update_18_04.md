---  
author: Rituraj Raman r.ramanbits@gmail.com  
date: 4th June 2020  
---  
  
## Updating your Ubuntu version from the command line  
  
My preferred method of moving to the next Ubuntu release has always been a fresh install. Old habits picked up from my windows days :D.  
I always used to think that a fresh install is the way to go, it should change all packages to the correct versions, and remove the bloatware I had accumulated over the 2-3 years of use. For the latter part, I have come to learn a lot about general ubuntu maintenance, as a result of better understanding of Debian package management mechanism.  
But along with a fresh install, also comes the pain of setting up a bootable USB, taking a complete backup of projects not maintained on [GitHub](https://github.com/) (or whichever remote version control you use), and a LOT of issues with the grub (or [grub2](https://help.ubuntu.com/community/Grub2)) on multi-boot systems. So when I came across the fact that it is, in fact, possible to very cleanly update the Ubuntu version without making major changes to the system configuration and without fearing for the hundreds of gigabytes of backup to be done, I was elated.  
As it turns out, Ubuntu has made it really easy to update to new LTS versions as they come out. Although we can also update to the development releases, I prefer not to do so, for the obvious reason that there is not a lot of support for these intermediate versions, and it can get cumbersome for casual users like myself.  
  
>If you are reading this article, I assume you already know about LTS and development releases. But for the sake of completeness, **LTS** releases are **Long Term  Support** releases, generally released every 2 years, and officially supported for 5 years. These are well suited for people who are not concerned with the latest applications and want to maintain a working system, without any hiccups, for a long period of time.  
>**Development** releases, on the other hand, are released generally every 6 months, and are supported for a minimum of 9 months. Good for people who like to experiment a lot, and those who want to be up-to-date on what the ubuntu community has to offer!  
  
  
******************************************************************************

Based on my recent experience of updating from Ubuntu 16.04.06 to 18.04.04 (desktop version), I have tried to put together a guide for people who want to find all the relevant information in one place.  
The same steps apply to the server version as well, though in that case, I strongly recommend you to test out the release you are targeting on a live USB.  
  
******************************************************************************

1. First, we need to **update all packages in our current ubuntu installation to the latest version**, so that package conflicts and upgrades are kept to a minimum. This step is crucial if you do not want to run into lots of conflicts and menus during the actual update.  

    Run the following on your command line:  
      
    >**sudo apt -y update**  
    >**sudo apt -y upgrade**  
    >**sudo apt -y dist-upgrade**  
    >**sudo apt autoremove**  
    >**sudo apt clean**  
      
    Though you might see these commands and think, "I run these regularly on my setup! What is the big deal?", I am a big proponent of actually **_knowing_** what each step does, so that you have a clear understanding of what is actually happening in the system.  
      
    Let us go through these commands quickly. I have put the man page definition along with my understanding of the same.  
      
    * **sudo apt -y update**:  
    It downloads the package lists from the repositories and "updates" them to get information on the newest versions of packages and their dependencies. It will do this for all repositories and PPA.   
    >From http://linux.die.net/man/8/apt-get:  
    >Used to re-synchronize the package index files from their sources. The indexes of available packages are fetched from the location(s) specified in /etc/apt/sources.list(5). An update should always be performed before an upgrade or dist-upgrade. If you do not run that command then you may get an out-of-date package installed.  
      
      
    * **sudo apt -y upgrade / sudo apt -y dist-upgrade**:  
    It will fetch new versions of packages existing on the machine if APT knows about these new versions by way of apt-get update. upgrade holds back packages if there are unresolved dependencies, or unmet dependencies; dist-upgrade intelligently handles the packages and their dependencies as well, so that all packages are correctly upgraded to the latest version.  
    >From http://linux.die.net/man/8/apt-get:  
    >upgrade  
    >Used to install the newest versions of all packages currently installed on the system from the sources enumerated in /etc/apt/sources.list(5). Packages currently installed with new versions available are retrieved and upgraded; under no circumstances are currently installed packages removed, nor are packages that are not already installed retrieved and installed. New versions of currently installed packages that cannot be upgraded without changing the install status of another package will be left at their current version. An update must be performed first so that apt-get knows that new versions of packages are available.  
    >    
    >dist-upgrade  
    >dist-upgrade in addition to performing the function of upgrade, also intelligently handles changing dependencies with new versions of packages; apt-get has a "smart" conflict resolution system, and it will attempt to upgrade the most important packages at the expense of less important ones if necessary. So, dist-upgrade command may remove some packages. The /etc/apt/sources.list file contains a list of locations from which to retrieve desired package files. See also apt_preferences(5) for a mechanism for overriding the general settings for individual packages.  
      
    **You can simply run apt dist-upgrade instead of apt upgrade. Running upgrade before dist-upgrade is a useless idiosyncrasy of mine :P**  
      
      
    There is also a command **apt full-upgrade**  
    >full-upgrade  
    full-upgrade performs the function of upgrade but may also remove installed packages if that is required in order to resolve a package conflict.  
      
    Since it removes packages, I recommend you do not use this unless absolutely necessary; and dist-upgrade is also complaining about broken packages.  
      
    * **sudo apt autoremove**:  
      
    Removes packages which are no longer needed. This is completely safe, and might prevent package conflicts because of different dependencies if you decide to install the same package later. Basically, apt autoremove removes the unneeded packages that were once installed as a dependency. When you uninstall a package, theses dependencies are of no use.  
    From http://manpages.ubuntu.com/manpages/xenial/man8/apt.8.html  
    >autoremove (apt-get(8))  
    >autoremove is used to remove packages that were automatically installed to satisfy dependencies for other packages and are now no longer needed as dependencies changed or the package(s) needing them were removed in the meantime. You should check that the list does not include applications you have grown to like even though they were once installed just as a dependency of another package. You can mark such a package as manually installed by using apt-mark(8). Packages which you have installed explicitly via install are also never proposed for automatic removal.  
      
      
    * **sudo apt clean**:  
    Whenever you install a new package, or upgrade an existing package (using apt install or apt (dist-)upgrade), the .deb files are downloaded from the repositories in /etc/apt/sources.list into var/cache/apt/archives/ directory and installed from there. Once installation is complete, you do not need these files, and they can be removed safely from the disk. Although this step makes no functional improvement, this is a good way to free up disk space and reduce clutter.  
  
******************************************************************************
  
2. Let us install the update-manager-core package now. It should already be available on 16.04 and above. This is needed for the next step, which is the actual upgrade. This package is in charge of taking care of all the changes which are to be done in order to update to the next release.  
  
    Run the following:  
    **sudo apt install update-manager-core**  
    >From: https://packages.ubuntu.com/bionic/admin/update-manager-core  
    >It manages release upgrades.   
      
    Now, just check /etc/update-manager/release-upgrades and ensure that the Prompt value is set to lts.  
    From the file:  
    >lts    - Check to see if a new LTS release is available.  The upgrader will attempt to upgrade to the first LTS release available after the currently-running one.  Note that if this option is used and the currently-running release is not itself an LTS release the upgrader will assume prompt was meant to be normal.  
    **So if you are on 14.04, it will still update to 16.04** (since it is officially supported till 2021). SO if you want to go from 14.04 to 16.04, either:  
    * Use the next step to update to 16.04, then repeat to go to 18.04, or better,  
    * Do a fresh install of 18.04 via USB after taking a full backup of the system.  
  	
******************************************************************************
  
3. Here comes the fun part! Let us now **_actually_** update our Ubuntu version!  
  
    Run:  
    **sudo do-release-upgrade**  
    This is the major command which actually does the upgrade to the next release, which has been chosen in /etc/update-manager/release-upgrades  
    From http://manpages.ubuntu.com/manpages/cosmic/en/man8/do-release-upgrade.8.html  
    Upgrade  the  operating  system  to the latest release from the command-line.  This is the preferred command if the machine has no graphic environment or if the  machine  is  to  be upgraded over a remote connection.  

    One caveat here is that if you are asked about grub update, choose "keep current configuration" on multi boot systems, else you might run into problems while booting, only ubuntu will be able to boot.  

******************************************************************************


Once the process is complete, you would be asked to restart the system, and there you have it! Enjoy running your shiny new OS!

Do email me at r.ramanbits@gmail.com if you have any issue with this guide, I would be happy to help! :)