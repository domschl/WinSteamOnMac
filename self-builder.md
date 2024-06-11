- 2024-06-03: ![Note:](http://img.shields.io/badge/🛑-Error-red.svg?style=flat) Unfortunately Apple's installation is currently broken when not using a 3rd-party pre-built toolkit. See main readme.
- 2024-03-24: ![Note:](http://img.shields.io/badge/⚠️-Warning-orange.svg?style=flat) Apple's `game-porting-toolkit` currently requires an older version of Apple's command line tool (version 15.1) in order to install successfully! Current Xcode 15.3 __will not work__!

-----------------------

### Built yourself __(currently broken)__

- If you want to try to build the toolkit yourself (currently broken!): Command Line Tools for Xcode **15.1** are required to be able to install `Game Porting Toolkit`. **WARNING** Apple's `game-porting-toolkit` currently fails to built with _all_ versions, so use a prebuilt toolkit for the time being.
  
> #### Get the correct command line tools (optional, only for self-builders, won't work anyway)
>
> Get the command line tools here [Command line tools 15.1](https://download.developer.apple.com/Developer_Tools/Command_Line_Tools_for_Xcode_15.1/Command_Line_Tools_for_Xcode_15.1.dmg)
>
> If you have Xcode concurrent to command line tools installed, you can use `xcode-select -p` to check which version is active. The correct output after installation of command line tools 15.1 should be: `/Library/Developer/CommandLineTools`. In case a path to Xcode is shown, you can can correct the path with:
>
> ```
> sudo xcode-select -s /Library/Developer/CommandLineTools`.
> ```

#### Continue with preparations
Now we install Apple's homebrew tap:

```bash
brew86 tap apple/apple http://github.com/apple/homebrew-apple
```

and install the toolkit:

```bash
brew86 -v install apple/apple/game-porting-toolkit
```

> **Note:** This starts a rather lengthy install- and compilation process that takes a long time! The compilation process (which can take an hour or more depending on your machine) finished with. It will compile ancient compiler versions required first and then setup a specific version of `wine` tools that allow Windows programs to run on macOS.

If the build is successful, you should see:

```
==> game-porting-toolkit
Please follow the instructions in the Game Porting Toolkit README to complete installation.
```

Instead, in order to continue the initial installation, skip down to **Continue with initial installation**

### Update notes

> **Note:** This section is only for updates to a new version, once the complete installation was finished successfully before.
> For initial installation, continue with **Continue with initial installation**.

- Make sure to use both the latest release of macOS Sonoma, the latest latest release of `game-porting-toolkit`, and precise version 15.1 of the Command Line Tools for Xcode. 15.1 is the only supported version!

If you want to update your x86 homebrew at a later point, start an x86 shell in Terminal with:

```bash
arch -x86_64 zsh
```

then in the x86 shell:

```bash
alias brew86=/usr/local/bin/brew
```

```bash
brew86 update
brew86 upgrade
```

This will update your x86 homebrew environment and automatically build the latest `game-porting-toolkit`.
Remember, the build process will take about 30min even on a very fast machine!

Then you will need to download the latest [Game Porting Toolkit](https://developer.apple.com/download/all/?q=game%20porting%20toolkit) (select the "Evaluation environment for Windows games 2.0 beta") or the game porting toolkit, which contains the former too, to update the libraries in your wine prefix. Open the download and:

```bash
ditto /Volumes/Evaluation\ Environment\ For\ Windows\ Games-2.0/redist/lib/ `brew --prefix game-porting-toolkit`/lib/
```

That's all that's needed for an update. Below information applies only to the initial installation.

When starting the Steam client after an update, remember that the initial first start might take several minutes (**patience!**), and is not always successful without 'retry': see below for common issues.

#### Update trouble

If steam doesn't start after an update:

- have _patience_! First startup of steam seems to take several minutes. Sometimes a huge error-box displayed. Click on the box to make sure it has active input focus and press `Enter` to make it vanish.
-  When asked, select 'restart Steam' as response to possible errors. In many cases, the steam client at some point does appear!

##### Last resort on failed updates

- try to re-install steam: `WINEPREFIX=~/Win10 /usr/local/opt/game-porting-toolkit/bin/wine64 SteamSetup.exe`, or:
- completely remove your homebrew x86 installation at `/usr/local/` and start over. (See below 'Uninstallation Notes' for proper removal.)

#### Problems after Steam Client self-update

- Self-update of steam client gets stuck. Wait for things to 'settle', then restart your computer to eliminate old rogue processes. It might be necessary to manually kill `wine64-preloader` instances before a restart is possible. After restart, the Steam Client update should work fine.

### Continue with initial installation

Now create a directory that will contain all the Windows stuff, Steam and any games you download. Here we'll use `~/Win10`,
which is our `wine` prefix (Apple uses the example of `my-game-prefix`, we'll go with `Win10`):

```bash
mkdir ~/Win10
WINEPREFIX=~/Win10 `brew86 --prefix game-porting-toolkit`/bin/wine64 winecfg
```

Note: we use the alias `brew86` we defined above.

<img src="https://github.com/domschl/WinSteamOnMac/blob/main/Resources/winecfg.png" width="30%" align="right">

This should start the `winecfg` program, a small setup for our `wine` environment. 

- In WineCfg, select `Windows 10`, and `Apply`, 

- Now open the Game Porting Toolkit which you downloaded above in Finder to verify it's mounted.

<img src="https://github.com/domschl/WinSteamOnMac/blob/main/Resources/gameporting.png" width="60%">

From your Terminal, it should be available at:

```bash
ls /Volumes/Evaluation\ Environment\ For\ Windows\ Games-2.0
```

Make sure you see the files of the toolkit and then:

```bash
ditto /Volumes/Evaluation\ Environment\ For\ Windows\ Games-2.0/redist/lib/ `brew --prefix game-porting-toolkit`/lib/```

This (silently) copies the required apple libraries into your `wine` installation. Note that since beta-4, the libs on the image from Apple are in `redist/lib` (older Beta versions: `lib`)

Now copy the Windows setup of steam (we assume you downloaded into the default `~/Downloads` folder) into your new Windows drive:

```bash
cp ~/Downloads/SteamSetup.exe ~/Win10/drive_c
```

Now enter your `Win10` directory and note the full path of this directory:

```bash
cd ~/Win10
pwd
```

It should say something like `/Users/you-user-name/Win10`. This is your `wine`-prefix, which you need to use below.

Apple provides a few scripts to start games, which we will not use: `gameportingtoolkit`, `gameportingtoolkit-no-esync`, and `gameportingtoolkit-no-hud`. Open those script with your favorite editor to learn how to configure `wine`. The `Readme.rtf` or the Game Porting Toolkit explains the differences.

We use what we learned from inspecting the scripts and directly start the Steam setup with:

```bash
cd ~/Win10/drive_c
MTL_HUD_ENABLED=0 WINEESYNC=1 WINEPREFIX=~/Win10 /usr/local/opt/game-porting-toolkit/bin/wine64 SteamSetup.exe
```

This should now start the Steam setup process. Be patient.

- If you get a failure dialog box with several options, as "Restart Steam?", try that, and on continued failure exit the dialog,
- Reboot(!)   (There seem to be rogue processes that prevent successful retries without reboot)
- And after reboot, open 'Terminal' and start Steam directly:

```bash
cd ~/Win10/drive_c
MTL_HUD_ENABLED=0 WINEESYNC=1 WINEPREFIX=~/Win10 /usr/local/opt/game-porting-toolkit/bin/wine64 Program\ Files\ \(x86\)/Steam/Steam.exe
```

This took some time (**patience!**), but then the Steam logon appears! Horray!

Login, and start installing your favorite Windows games...

### Quality of life enhancement.

We can use Apple's 'Shortcuts' app to automate the process of launching Wine & Steam Windows. Start the Shortcuts app, create a new shortcut and add the action 'Run Shell Script'.

![](Resources/shortcuts.png)

The script text is:

```bash
cd ~/Win10/drive_c
MTL_HUD_ENABLED=0 WINEESYNC=1 WINEPREFIX=~/Win10 /usr/local/opt/game-porting-toolkit/bin/wine64 Program\ Files\ \(x86\)/Steam/Steam.exe
```

> **Note:** 'Shortcut' might ask you for confirmation, if you want to run scripts, and you need to enable that via the settings-link provided.

Add an icon to the shortcut, and you are ready to go. The shortcut can be put into the dock, and now you simply can directly start Steam for Windows! 
(Right-click the shortcut script in the 'Shortcuts' app and select 'Add to dock' to create shortcut for your shortcut in the Shortcuts app...)

### Closing Steam

- Use Steamclient's GUI menu (Steam / Exit) or the menu bar icon (right-click, 'Exit Steam') to terminate steam. The GUI should display 'Shutting down Steam', indicating a proper termination of all Steam processes.
- Just Ctrl-C-ing in the terminal (or other methods to 'suddenly' terminate Steam will cause crash-bug-workarounds to kick in, restarting Steam automatically again and again...

### Troubleshooting

- See the `Readme.rtf`, section 'Troubleshooting' in Apple's `game-porting-toolkit` for advice on problems with specific games.
- In case you got stuck somewhere during install, follow the 'Uninstallation notes' below before retrying a new installation.

### Uninstallation notes

- Uninstall the Intel version of homebrew, enter a x86 shell with `arch -x86_64 zsh`, following [this link to the uninstall-script](https://github.com/homebrew/install#uninstall-homebrew). There are three different deinstall scripts listed, you'll want the last one that selects the specific homebrew x86 version at `/usr/local`. Use the last option that allows selecting the prefix `/usr/local` to make sure that you remove the correct Homebrew version. Execute this script from a X86-64 Shell created with `arch -x86_64 zsh`.
- Simply remove the directory tree that holds your wine-prefix: remove `~/Win10`.
- Note: If you want to reinstall, make sure to _reboot once_ to remove any remaining steam or setup processes from memory.
