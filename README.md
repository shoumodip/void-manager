# Void manager
NixOS inspired system management in Void linux. Basically I was blown away by the system management in Nix but was way too lazy to distro hop. Therefore I came with this script which does what I *really* liked about NixOS. ***NOTE:*** This does not incorporate even 90% of the features of the Nix system. Declarative package management and generations is way too complex for a shell script, and I couldn't be bothered to create an entire parser system in C or Haskell or whatever. This script incorporates the basic features of Nix, features which *I* wanted in my system.

## Dependencies
- Base install of void linux (that's the minimal requirement)
- Non-privileged user

## Installation
```sh
git clone https://www.github.com/shoumodip/void-manager
cd void-manager

sudo mv vman /usr/local/bin/
mv config.void ~/.config/
```

## Features
- Package list management
- Post rebuild run hooks
- Post initial-build hooks
- Powerful and full featured configuration language (`bash` lol)
- Autologin user management for `tty1`
- Custom authentication systems.

## Why void manager
If it hasn't happened already to you, let me make a little prophecy. There will come a time when you will start looking into automation scripts for getting setup in any system with all your programs and dotfiles already present and configured. You will probably create a script which installs your programs and symlinks your dotfiles and gets you ready in any system whatsoever. That's where the problems start. Syncing. Every single time you install a package on your system and you like it, you need to manually put it in the installer script. However there *will* come a time when you miss out on some programs which are used in your dotfiles and scripts. And you will have a major misfunctioning at the absolute worst moment possible. This is where void manager comes into play. It will make dotfile management and package list control ***sane***.

## Features
Void manager looks for the configuration file `~/.config/config.void` by default. All the examples shown below belongs in the config file.

### Packages
The elephant in the room. When void is being built or rebuilt, it will install all the items in the `packages` list. Not installed packages will get installed, outdated packages will get updated, all in one go. All well and good. Now the magic of this starts. If you have a package which you *explicitly* told Xbps to install, via `xbps-install` or previously installed by void, if the package is not listed now, it will get **removed**. Think about that. If you have packages you are not supposed to have according to your configuration, they will get removed. This already fixes all the problems with program lists gettings out of sync. If you have packages on one system, it will synced across *all* your setups.

Now before you start fretting, note. It will remove *manually installed* not-listed packages **only**. It won't remove the packages which got installed by Xbps as the dependencies of a package you told it to install. So there is no need to list *all* the dependencies of a package in the fear of them getting removed. For more information on this, I highly recommend you to check out the `-m` flag in the Xbps documentation. (`man xbps-query`)

An example packages list.

```sh
packages=(
  # Sound
  "pulseaudio"
  "pavucontrol"

  # Tools
  "geany"
  "firefox"

  # GUI
  "xorg"
  "xfce4"
  "xfce4-pulseaudio-plugin"
)
```

### Post rebuild hook
Every single time you rebuild your system using `vman` or `vman rebuild`, the commands listed here will get executed. So if you want a particular thing to happen every time you refresh your packages or whatever, you can do it!

This can be really useful. For example let's say you want your dotfiles to be pushed into the remote git repository whenever you rebuild your system. With the rebuild hook you can make it happen.

```sh
execute=(
  "notify-send 'Void Manager' 'Your system has been put back in sync!'"

  # Automatic dotfile syncing with git after each rebuild of void
  "cd dotfiles && git add . && git commit -m 'Synced void' && git push -u origin main"
)
```

### Post build hook
Same as above but a little different. The commands listed here will be run only *once*, after your system has been built using void-manager for the first time. Place your dotfiles symlinking commands here, as they only need to happen once when your system is being setup. Think of this as the equivalent to the custom script you wrote.

```sh
execute_once=(

  # BTW, this is how you do authenticated operations, Use the `$auth` variable
  "$auth ln -sf /etc/sv/dbus/ /var/service/"

  # The sweet sweet emoji support through your custom script or whatever
  "patch-libxft"

  # Dotfiles
  "git clone https://www.github.com/USERNAME/dotfiles"
  "ln -s dotfiles/* ~/"
)
```

Isn't that neat!

### Auto login in `tty1`
This is an optional and highly opinionated setting. I don't like to put in the password every single time I start my computer. And there's no security risk with disabling it *for me*. My friends are windows users, (Yes **users**, not *normies* or *plebs*. Stop it with the damn evangelism, linux users! Ugh) so there is literally no risk there. The window manager I use (DWM) will prevent them from getting access to my stuff (memes and vim plugins lol). Therefore I have literally no use for the authentication at the `tty1`. Therefore I put this in as an *option*.

```sh
# In case it isn't obvious, replace USERNAME with an actual user name
autologin=USERNAME
```

You can put in a username of your choice there and after rebuilding your void configuration, said user will get "auto-loggined" in the `tty1` virtual console. Again, *only* use this if you are sure there will be no security repurcussions from this. If someone gets a hold on your data because you disabled the `tty1` authentication, don't blame me!

### Custom authentication system
This is a setting most users don't need to touch. Yet I put it in because I know some users (\*cough\* me \*cough\*) like to use programs like `doas` in place of `sudo`. There are a lot of reasons why one might do that, overflow exploit issues with `sudo`, `doas` being easier to use or just to look cool on the internet (\*nervous sweating\* yes?).

```sh
# Dab on the sudo users
autologin=doas
```

## How do I get started?
So you decided to give void manager a try, cool. Here are the things you should do.

- Install void manager (duh)
- Run `vman help` in the terminal. See the options of void and learn what they do
- Open the configuration file and read the comments if you feel unsure about something
- Run `xbps-query -m` and go through the packages you have manually installed
- Put the packages you want your system to have in your configuration file
- BTW, no need to put in `grub` or `base-system` or `sudo` in your packages list
- Put the dotfiles symlinking and the system setup commands in the `execute_once` list
- Run `vman` or `vman build` or `vman start` (It's recommended to use `vman` only)
- Enjoy the fully-synced partially-declarative approach to system management

## License
MIT
