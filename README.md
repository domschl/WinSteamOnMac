# Howto install the Windows version of Steam on macOS Sonoma

We will install the x86 version Homebrew in order to be able to use Apple's modified version of `Wine` and to install the Windows version Steam.

We will make sure that our existing environment (and the Apple silicon version of Homebrew we need for 'serious' work) remains undisturbed.

<img src="https://github.com/domschl/WinSteamOnMac/blob/main/Resources/WindowsSteamOnMac.png" width="50%" align="right">

This guide is only tested for Apple Silicon machines.

## Latest tested versions

- 2024-06-11: ![Note:](http://img.shields.io/badge/🛑-Error-red.svg?style=flat) New version 2.0 beta available, but compilation of the toolkit _still fails_, which is not exactly suprising, since the homebrew formula hasn't been updated, and still uses the broken 1.1 version. The current work-around is to install a pre-built version of the toolkit.
- 2024-06-03: ![Note:](http://img.shields.io/badge/🛑-Error-red.svg?style=flat) Unfortunately Apple's installation is currently broken. This guide doesn't work until the cause has been identified and fixed! See [Issue 9](https://github.com/domschl/WinSteamOnMac/issues/9) for details.
- 2024-03-24: ![Note:](http://img.shields.io/badge/⚠️-Warning-orange.svg?style=flat) Apple's `game-porting-toolkit` currently requires an older version of Apple's command line tool (version 15.1) in order to install successfully! Current Xcode 15.3 __will not work__!

## Preparations:

- Go to [Apple Games](https://developer.apple.com/games/) in order to download the [Game Porting Toolkit](https://developer.apple.com/download/all/?q=game%20porting%20toolkit). There are two possible downloads: "Evaluation environment for Windows games 2.0 beta" and "Game porting toolkit 2.0 beta" which is larger and contains the 'Evaluation environment' too. We will need the "Evaluation environment for Windows games 2.0" only.
- If you want to try to build the toolkit yourself (currently broken!): Command Line Tools for Xcode **15.1** are required to be able to install `Game Porting Toolkit`. **WARNING** Apple's `game-porting-toolkit` __fails to build with current Xcode or Command line tools newer than 15.1, you **must** use the older version 15.1__ to be able to build the toolkit successfully. You can decide to keep the recent Xcode, and install the older command line tools additionally and use `xcode-select -s` to switch between versions.

> #### Get the correct command line tools (optional, only for self-builders)
>
> Get the command line tools here [Command line tools 15.1](https://download.developer.apple.com/Developer_Tools/Command_Line_Tools_for_Xcode_15.1/Command_Line_Tools_for_Xcode_15.1.dmg)
>
> If you have Xcode concurrent to command line tools installed, you can use `xcode-select -p` to check which version is active. The correct output after installation of command line tools 15.1 should be: `/Library/Developer/CommandLineTools`. In case a path to Xcode is shown, you can can correct the path with:
>
> ```
> sudo xcode-select -s /Library/Developer/CommandLineTools`.
> ```

#### Continue with preparations

The "Evaluation environment for Windows games 2.0" (contained in Game Porting Toolkit 2 or downloaded stand-alone) contains a Readme that outlines the installation process, but here we will customize it, in order to run Steam.

- [Download Steam](https://store.steampowered.com/about/download). Make sure to download the [Windows setup](https://cdn.akamai.steamstatic.com/client/installer/SteamSetup.exe), and not the (default) Mac version. You should now have a file `SteamSetup.exe`.

## Step-by-step installation

**Note:** See `Update notes` below, if you did already install an older version!

- The minimum macOS version is macOS Sonoma 14.5, Command Line Tools 15.1 (newer versions of command line tool are __not supported__), Game Porting Toolkit 2
- This guide only applies to Apple Silicon Macs. No Intel support.
- Open a terminal (or iTerm2)
- Make sure that rosetta is installed by entering:

```bash
softwareupdate --install-rosetta
```

Now your Mac is able to execute x86_64 code. This is the basis for all the following installation.

- Now switch to a x86 shell by entering:

```bash
arch -x86_64 zsh 
```

Type `arch` again, to make sure that you are using Intel. It should **not** show `arm64`, but `i386` ( ;-) )

Now, from a terminal that uses `x86_64` arch, install homebrew for x86:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

This will install the Intel x86 version of homebrew to `/usr/local`. If you already installed homebrew
for Apple Silicon, then that version resides in `/opt/homebrew` and won't be touched. This guide will assume that the Apple Silicon
homebrew is your important version, and will remain the default when working with terminal or using `brew`. If you do not have
an Apple Silicon version of homebrew installed, don't worry, nothing we do here requires that or modifies any of it.

Do **not** follow the recommendation at the end of the x86-homebrew install script to put `shellenv` into `.zprofile`. (That 
would put the x86 Version of Homebrew into your paths, conflicting with an Apple Silicon version of homebrew. No paths or
environment modifications are needed in order to proceed!)

In order not to mess up the two homebrew versions, we create an alias for the Intel homebrew:

```bash
alias brew86=/usr/local/bin/brew
```

Note: if you are following Apple's readme, make sure to replace all instances of `brew` in Apple's doc with `brew86` from now on.

## Build yourself or use pre-built toolkit

### Pre-built toolkit

Use Dean Greer's (GCenX) versions of the toolkit that have been prebuilt. This is the faster installation method (and currently the only non-broken one):

```bash
brew86 install --cask --no-quarantine gcenx/wine/game-porting-toolkit
```

This installs a graphical Application "Game Porting Toolkit" based on the old working binaries that opens a pre-configured terminal with all the tools. Go to applications and open "Game Porting Toolkit". In the terminal window that gets started by the application, enter:

`wine winecfg`

To verify everything is working. Close Winecfg and start with the update procedure to the latest drivers.

Make sure that you have opened the "Evalutaion environment for Windows Games". You should see this at `/Volumes/Evaluation environment for Windows games 2.0`. Then start the update:

```bash
cd /Applications/Game\ Porting\ Toolkit.app/Contents/Resources/wine/lib/external
mv D3DMetal.framework D3DMetal.framework-old; mv libd3dshared.dylib libd3dshared.dylib-old
ditto /Volumes/Evaluation\ environment\ for\ Windows\ games\ 2.0/redist/lib/external/ .

This is silent on success.

Now you are ready to install Steam:

```bash
MTL_HUD_ENABLED=0 WINEESYNC=1 wine ~/Downloads/SteamSetup.exe
```

After some time, the Steam login appears!

Steam has been installed into the wine prefix at `~/.wine` and your Toolkit software resides within the application "Game Porting Toolkit". From any Terminal, you can now start Steam with:

```bash
MTL_HUD_ENABLED=0 WINEESYNC=1 /Applications/Game\ Porting\ Toolkit.app/Contents/Resources/wine/bin/wine ~/.wine/drive_c/Program\ Files\ \(x86\)/Steam/steam.exe
```

This takes some times, but then Steam starts.

-----------------------

### Built yourself __(currently broken)__

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

### References

- Good collection of information: [Apple Gaming Wiki](https://www.applegamingwiki.com/wiki/Game_Porting_Toolkit)
- A 'batteries-included' GUI version to get the game-porting-toolkit running: [Whisky](https://github.com/Whisky-App/Whisky)
- Pre-built versions of the toolkit are available at [Gcenx repository](https://github.com/Gcenx/game-porting-toolkit/releases)

### History

- 2024-06-11: Apple releases `game_porting_toolkit` 2.0 beta 1.
- 2024-03-26: Apple's `game_porting_toolit` requires Command Line Tools version 15.1 to build successfully, newer versions are not (yet) supported.
- 2024-03-24: Apple's `game_porting_toolkit` is broken, the project currently doesn't install until the build is fixed by Apple.
- 2024-03-08: macOS 14.4 testet ok.
- 2023-11-24: Apple Game Porting Toolkit 1.1 changes. Addressed suggestions by @mobigroup and @cdtj concerning username placeholders.
- 2023-10-06: New installations with Xcode 15, Sonoma 14.0 release and `game-porting-toolkit` 1.0 retested, ok.
- 2023-10-06: Small fixes for release version of `game-porting-toolkit` 1.0, Uninstallation and troubleshooting notes.
- 2023-09-26: macOS Sonoma 14.0 Release tested ok. No changes.
- 2023-09-24: Retest with Sonoma RC, `game-porting-toolkit` Beta 4 (Note: library path on Apple's IMG has changed from `lib` to `redist/lib`.
- 2023-08-14: Updates for `game-porting-tookit` Beta 3 alias 1.0.3 and Sonoma Beta 5. (No significant changes to the update procedure).
- 2023-07-04: Section **Update notes** added. Apple has published a new version 1.0.2 of the game-porting toolkit.
