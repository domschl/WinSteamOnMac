# Howto install the Windows version of Steam on macOS Sonoma Beta

We will install the x86 version Homebrew in order to be able to use Apple's modified version of `Wine` and to install the Windows version Steam.

We will make sure that our existing environment (and the Apple silicon version of Homebrew we need for 'serious' work) remains undisturbed.

<img src="https://github.com/domschl/WinSteamOnMac/blob/main/Resources/WindowsSteamOnMac.png" width="50%" align="right">

## Preparations:

- Go to [Apple Games](https://developer.apple.com/games/) in order to download the [game porting toolkit](https://developer.apple.com/download/all/?q=game%20porting%20toolkit)

This contains a Readme that outlines the process, but here we will customize it, in order to run Steam.

- [Download Steam](https://store.steampowered.com/about/download). Make sure to download the [Windows setup](https://cdn.akamai.steamstatic.com/client/installer/SteamSetup.exe), and not the (default) Mac version. You should now have a file `SteamSetup.exe`.

## Step-by-step installation

- You **must** run the Beta of macOS Sonoma!
- game-porting-toolkit 1.0.2 requires at least beta 3 or Sonoma.
- Open a terminal (or iTerm2)
- Make sure that rosetta is installed by entering:

```bash
softwareupdate --install-rosetta
```

Now your mac is able to execute x86_64 code. This is the basis for all the following installation.

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
for Apple Silicon, then this version resides in `/opt/homebrew`. This guide will assume that the Apple Silicon
homebrew is your important version, and will remain the default.

In order not to mess up the two homebrew versions, we create an alias for the Intel homebrew:

```bash
alias brew86=/usr/local/bin/brew
```

Note: if you are following Apple's readme, make sure to replace all instances of `brew` in Apple's doc with `brew86` from now on.

Now we install Apple's homebrew tap:

```bash
brew86 tap apple/apple http://github.com/apple/homebrew-apple
brew86 -v install apple/apple/game-porting-toolkit
```

Now create a directory that will contain all the Windows stuff, Steam and any games you download. Here' we'll use '~/Win10',
which is our `wine` prefix (Apple uses the example of `my-game-prefix`, we'll go with `Win10`):


### Update notes

**Note:** This section is only for updates later, once the complete installation was finished. For initial installation, continue with **Continue installation**.

**Note:** Update doesn't seem to be a straight foreward process. You might need to completely start over, if things go wrong. 

- `game-porting-toolkit` version 1.0.2 seems to require at least beta 3 of macOS Sonoma.

If you want to update your x86 homebrew at a later point:

```bash
arch -x86_64 zsh
alias brew86=/usr/local/bin/brew
brew86 update
brew86 upgrade
```

This will update your x86 homebrew environment and automatically build the latest `game-porting-toolkit`.

Then you will need to download the latest [game-porting-toolkit](https://developer.apple.com/download/all/?q=game%20porting%20toolkit) from apple to update the libraries in your wine prefix. Open the download and:

```bash
ditto /Volumes/Game\ Porting\ Toolkit-1.0/lib/ `brew86 --prefix game-porting-toolkit`/lib/
```

That's all that's needed for an update. Below information is only initial install.

If steam doesn't start after an update, you can either

- try to re-install steam: `WINEPREFIX=/Users/your-user-name/Win10 /usr/local/opt/game-porting-toolkit/bin/wine64 SteamSetup.exe` (replace `your-user-name`), or:
- completely remove your homebrew x86 installation at `/usr/local/` and start over.

### Continue installation

```bash
mkdir ~/Win10
WINEPREFIX=~/Win10 `brew86 --prefix game-porting-toolkit`/bin/wine64 winecfg
```

Note: we use the alias `brew86` we defined above.

<img src="https://github.com/domschl/WinSteamOnMac/blob/main/Resources/winecfg.png" width="30%" align="right">

This should start the `winecfg` program, a small setup for our `wine` environment. Select `Windows 10`, and `Apply`, 

MTL_HUD_ENABLED=0 WINEESYNC=1 WINEPREFIX=/Users/dsc/my-game-prefix /usr/local/opt/game-porting-toolkit/bin/wine64 Program\ Files\ \(x86\)/Steam/Steam.exe

- Now open the Game Porting Toolkit which you downloaded above.

<img src="https://github.com/domschl/WinSteamOnMac/blob/main/Resources/gameporting.png" width="60%">

From your Terminal, it should be available at:

```bash
ls /Volumes/Game\ Porting\ Toolkit-1.0/
```

Make sure you see the files of the toolkit and then:

```bash
ditto /Volumes/Game\ Porting\ Toolkit-1.0/lib/ `brew86 --prefix game-porting-toolkit`/lib/
```

This copies the required apple libraries into your `wine` installation.

Now copy the Windows setup of steam (we assume you downloaded into the default ~/Downloads` folder) into your new Windows drive:

```bash
cp ~/Downloads/SteamSetup.exe ~/Win10/drive_c
```

Now enter your `Win10` directory and note the full path of this directory:

```bash
cd ~/Win10
pwd
```

It should say something like `/Users/you-user-name/Win10`. This is your `wine`-prefix, which you need to use below (instead of `your-user-name` placeholder).

Apple provides a few scripts to start games, which we will not use: `gameportingtoolkit`, `gameportingtoolkit-no-esync`, and `gameportingtoolkit-no-hud`. Open those script with your favorite editor to learn how to configure `wine`. The `Readme.rtf` or the Game Porting Toolkit explains the differences.

We use what we learned from inspecting the scripts and directly start the Steam setup with:

```bash
cd ~/Win10/drive_c
MTL_HUD_ENABLED=0 WINEESYNC=1 WINEPREFIX=/Users/your-user-name/Win10 /usr/local/opt/game-porting-toolkit/bin/wine64 SteamSetup.exe
```

Make sure to replace `your-user-name` above with the output of the `pwd` command.

This should now start the Steam setup process. Be patient.

For me, it stayed with a black windows with no content for quite some time. After waiting patiently, I aborted the setup (be sure not to abort too early!), and started Steam directly:

```bash
MTL_HUD_ENABLED=0 WINEESYNC=1 WINEPREFIX=/Users/your-user-name/Win10 /usr/local/opt/game-porting-toolkit/bin/wine64 Program\ Files\ \(x86\)/Steam/Steam.exe
```

This again took some time, but then the Steam logon appears! Horray!

Login, and start installing your favorite Windows games...

### Quality of life enhancement.

We can use Apple's shortcuts app to automate the process of launching Wine & Steam Windows. Start the shortcuts app, create a new shortcut and add the action 'Run Shell Script'.

![](Resources/shortcuts.png)

The script text is:

```bash
cd ~/Win10/drive_c
MTL_HUD_ENABLED=0 WINEESYNC=1 WINEPREFIX=/Users/your-user-name/Win10 /usr/local/opt/game-porting-toolkit/bin/wine64 Program\ Files\ \(x86\)/Steam/Steam.exe
```

Again, make sure to replace `your-user-name` with the correct value!

Add an icon to the shortcut, and you are ready to go. The shortcut can be put into the doc, and now you simply can directly start Steam for Windows!

### History

2023-07-04: Section **Update notes** added. Apple has published a new version 1.0.2 of the game-porting toolkit.
