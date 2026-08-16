# ZERENE OS DOCUMENTATION / GUIDANCE

Zerene OS is a reproducible hybrid imperative-declare Linux distribution designed  for init freedom

This guide is going to teach you how to have use Zerene OS and be comfortable with it as you use it.

This guide already assumes you know how to set up xorg and stuff like this, so it will not explain how to, this guide strictly explain how the system works.

<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/a845ebd6-8445-4486-adb5-d05146d3d7f1" />




# HOW TO INSTALL

First, you need to boot into  Zerene OS, your machine needs to be an UEFI x86_64 machine with +4 gigs of ram to boot the iso, that is because the iso is incredibly compressed and stores itself entirely in ram.

<img width="752" height="423" alt="image" src="https://github.com/user-attachments/assets/318e6c21-b3cb-47d3-8a49-5974e35e47f2" />

After booting, you have to login as root.
password is root 

<img width="752" height="423" alt="image" src="https://github.com/user-attachments/assets/dc879fb5-8cb0-4482-9b34-cfa2e4960617" />


To install Zerene OS, you only need 2 commands :

– cfdisk (to set partitions)
– zstrappa (to bootstrap)

Using cfdisk, you will create 2 partitions

a boot drive of preferably 1 or 2 gb
and a root drive that covers the whole drive (or not if zerene is dualbooted

<img width="752" height="423" alt="image" src="https://github.com/user-attachments/assets/ea75d97b-8b80-43ce-94da-919f67e0ef14" />

On this example image, /dev/sda1 is my root drive
and /dev/sda2 is my boot drive.

The next command will bootstrap my Zerene OS system.

zstrappa /dev/sda1 /dev/sda2

<img width="752" height="423" alt="image" src="https://github.com/user-attachments/assets/89d23639-6dd2-45dd-ae6d-8f38aa6e0d3a" />

If you see this, that means Zerene OS is installing, wait until it finishes , and reboot,

You must immediately type “genkernel” after install to update your kernel.

Congrats! you have installed Zerene OS!
To learn how to actually use it, continue on this readme


# PACKAGE MANAGEMENT

Zerene OS has it’s own package manager called zeta, you can use it to install things both “imperatively” or “declaratively”

Zeta is written in Lua and only requires a standard Lua interpreter to run 
no special build tools or system dependencies.

– How to Use Zeta

Installing Packages

# Install a package from the online repository

zeta -Provide hello

<img width="752" height="422" alt="image" src="https://github.com/user-attachments/assets/fdfab35c-6e11-4409-89b3-e8c317f9b9a0" />

# Install multiple packages at once
zeta -Provide glib libffi

# Skip the confirmation prompt
zeta -Provide hello --pass
When you install a package, Zeta automatically pulls in all its dependencies too. For example, installing glib will also install libffi and pcre2 if they're needed and not already present.

Reinstalling Packages
# Force reinstall an already-installed package

zeta -ReProvide hello

Use -ReProvide when you want to overwrite a package that's already installed (e.g., after a corrupted download or to apply a newer version from the same URL).

Installing from Local Packages

If you have a local copy of packages (for offline use or development):
# Install from the local /packages tree

zeta -LocalProvide hello

# Zeta will try local first, then fall back to remote if a dependency is missing
Removing Packages

# Remove a single package

zeta -Remove hello

# Remove multiple packages

zeta -Remove hello libz

# Also remove dependencies that are no longer needed by anything

zeta -Remove hello --with-deps

# Override safety checks (skip "still required by" warnings)
zeta -Remove hello --force

Zeta is careful about removal:

- It warns you if a package you're removing is still needed by other installed packages

- It blocks removal of packages that other packages depend on (unless you use --force)

- Orphaned dependencies (no longer needed by anything) are removed silently or with --with-deps

Listing Installed Packages

zeta -List

This shows two sections: PACKAGES (things you explicitly installed) and DEPENDENCIES (things pulled in automatically). Each entry shows the name, version, and what it depends on.
Searching the Repository
# Search for packages matching a query

zeta -Localize wayland
zeta -Localize hypr

This searches the remote repository index by package name, summary, and version. Results are displayed as a table.
Testing Packages Offline

# Verify a local package builds correctly without installing it
zeta -Test hello
zeta -Test libz

This runs the package's full build pipeline into a scratch directory, runs its test hook (if any), and then throws everything away. Nothing is installed or recorded in the database. It's purely offline validation.
Package Sources

Zeta supports two package sources:

Remote Repository (default)
The default repository is hosted on GitHub:
https://github.com/gretagen/zeta-packages

Package manifests and tarballs are fetched from here. GitHub URLs are automatically rewritten to raw.githubusercontent.com for direct file access.

Local Package Tree

A directory of packages you maintain yourself, useful for:
- Offline installations
- Custom/internal packages
- Development and testing

The default location is /usr/share/packages or the packages/ directory next to the Zeta script.

Environment Variables

You can customize Zeta's behavior without editing any config files:
Variable    What it does    Default
ZETA_ROOT    Where packages are installed (acts as a chroot)    /
ZETA_REPO    Remote repository URL    https://github.com/gretagen/zeta-packages
ZETA_LOCAL_PACKAGES    Local package directory    /usr/share/packages
ZETA_CACHE    Download cache location    /var/cache/zeta
ZETA_STATE    Package database location    /var/db/zeta
ZETA_TMP    Temporary build area    /var/tmp/zeta
The ZETA_ROOT variable is especially useful for testing -- point it at a temporary directory and Zeta will install everything there instead of on your real system.
Safety Features
Sandbox
Every package's package.lua manifest runs in a sandboxed Lua environment. The package code cannot read files, write files, run shell commands, or access the network on its own. It can only declare what it wants to install. The actual installation happens through Zeta's own audited code.
Init-System Agnostic
Zeta never touches init system configuration. It refuses to install files under:
- etc/systemd/, usr/lib/systemd/
- etc/init.d/, etc/rc.d/, etc/init/
- etc/runlevels/
It also refuses to install files that claim distribution identity:
- /etc/os-release, /etc/lsb-release
- /etc/rc.conf, /etc/rc.local, /etc/inittab
This keeps Zeta compatible with any Linux distribution.
File Conflict Detection
Before installing, Zeta checks if any files you're about to install are already owned by another package. If there's a conflict, it stops and tells you which package owns the file. Use --force to override this.
Checksum Verification
Every package with a remote URL must declare a SHA-256 checksum. Zeta verifies the downloaded file matches before installing. If the checksum doesn't match, the install is aborted.
Atomic Database Writes
Package state is written atomically (write to a temporary file, then rename). If Zeta is interrupted during an install, the database is never left in a half-written state.
The p Object (for Package Authors)
When you write a package.lua with a custom build or install function, Zeta gives you a p object. This is your only interface to the system:
Method    What it does
p:run(cmd)    Run a shell command (like make or gcc)
p:cd(dir)    Change to a different directory
p:fetch(url)    Download a file and verify its checksum
p:unpack(archive)    Extract a tarball
p:install(src, dest)    Copy a file into the install root
p:meson(...), p:ninja(...), p:make(...), p:cmake(...)    Run common build tools
p:env_set(k, v)    Set an environment variable for commands
p:log(msg)    Print a log message
The p object tracks directory changes, so p:run("cd build && ninja") followed by p:ninja() will work correctly.
Example Packages
Simple binary install (hello)
return {
  name = "hello",
  version = "1.0",
  summary = "A tiny demonstration package",
  url = "hello-1.0.tar.gz",
  sha256 = "7318875...",
  archive = { strip = 1 },
}
Binary install with dependencies (glib)
return {
  name = "glib",
  version = "2.88.1",
  url = "https://github.com/.../glib-2.88.1.tar.xz",
  sha256 = "c6a04b3...",
  deps = { "libffi>=3.4", "pcre2>=10.42" },
  archive = { strip = 1 },
}
Build from source (libffi)
return {
  name = "libffi",
  version = "3.4.6",
  url = "libffi-3.4.6.tar.gz",
  sha256 = "1058fdc...",
  build = function(p)
    p:run("meson setup build --prefix " .. p.prefix)
    p:env_set("DESTDIR", p.install_root)
    p:ninja("-C", "build")
    p:ninja("-C", "build", "install")
  end,
}
Package with test hook (hyprland)
return {
  name = "hyprland",
  version = "0.56.2",
  url = "hyprland-0.56.2.tar.gz",
  sha256 = "e42923d...",
  deps = { "hyprutils" },
  archive = { strip = 1 },
  test = function(p)
    p:run("test -x '" .. p.install_root .. "'/usr/bin/Hyprland")
    p:run("test -x '" .. p.install_root .. "'/usr/bin/hyprctl")
  end,
}

Quick Reference
Command    What it does
zeta -Provide <pkg>...    Install packages from the remote repository
zeta -ReProvide <pkg>...    Reinstall packages (even if already installed)
zeta -LocalProvide <pkg>...    Install from the local package tree
zeta -Remove <pkg>...    Remove installed packages
zeta -List    Show installed packages and dependencies
zeta -Localize <query>    Search the remote repository
zeta -Test <pkg>    Verify a package offline without installing
zeta -Help    Show help
Flag    What it does
--pass    Skip confirmation prompts
--force    Override file conflicts and dependency safety checks
--with-deps    Also remove orphaned dependencies when removing a package

# Generations


<img width="752" height="423" alt="image" src="https://github.com/user-attachments/assets/c6d3c3de-c46e-4e71-89b7-6e314c262452" />

Zerene OS generations is a system that allows you to rollback in case you break your Zerene OS system

creating and managing Generations are allowed by using the “genzee”  command

Generations are created at the beginning of sync by the declarative configuration system so you always have a chance to rollback.

Generations are listed inside a subvolume of your limine bootloader so you can boot into them

When booting into a generation, they are strictly read only.
If you want to make a generation of the system your main boot entry again, you need to “rollback” to it.









To create a generation; use the “genzee create” command


<img width="752" height="423" alt="image" src="https://github.com/user-attachments/assets/8c7cf6f2-0839-4f91-b153-8690d1b2b36e" />

To list installed generations, use “genzee list”

<img width="752" height="423" alt="image" src="https://github.com/user-attachments/assets/e9dcbf3d-fccc-4350-b7b3-ce93c33b3af1" />

To remove a generation, use the “genzee remove” command
<img width="752" height="423" alt="image" src="https://github.com/user-attachments/assets/59b100c3-0f89-461c-866a-2e9176157c13" />


If your system breaks, boot into a generation , and rollback by using the command : “genzee rollback [number]

So if you want to rollback to generation 3, then use “genzee rollback 3”
and then reboot to your main zerene os system, your system will be like how it was when it was generation 3 
<img width="752" height="423" alt="image" src="https://github.com/user-attachments/assets/71fa9a97-f33f-424c-89e4-5d20346c2112" />

After rollback, you may use “genzee cleanup-next” to clean up. 

# DECLARATIVE CONFIGURATION

(THE SYSTEM IS HYBRID, IF YOU WANT TO USE ZERENE OS IMPERATIVELY, PLEASE DO NOT EDIT THIS FILE AND IGNORE THIS CHAPTER)

Like NixOS, Zerene OS can be declaratively configured with it’s own file, allowing reproducibility and easier system management.

To configure Zerene OS declaratively, you need to edit /etc/zerene/definition.lua

After editing, run “zerene-synchronize” to save your changes and make your system follow the definition.

<img width="752" height="423" alt="image" src="https://github.com/user-attachments/assets/99bba33b-d377-4a5c-a8f4-db28517ded3f" />

As seen, the system is configured in lua.

Here is what you can declare inside Zerene OS




– System Identity
Hostname, timezone, locale, keyboard, console font)

– Boot	
Kernel parameters, Limine bootloader settings (timeout, wallpaper, serial)

– Init System	
Choose between openrc or runit (experimental, will break things)

– Environment	
System-wide environment variables (EDITOR, BROWSER, etc.)

– Users	
Unix users to create/remove, with groups, shells, and SSH keys

– Packages	
Packages managed by Zeta -- listed = installed, unlisted = removed

–Subspaces	
Lightweight Linux environments (Debian, Arch, Alpine, etc.) via bubblewrap (see in “susbpaces” section)

– Kernel Modules	
Modules to load at boot

– Services	
Services to enable in the default runlevel

– Networking	
NetworkManager connection profiles (WiFi, Ethernet, static IPs)

– Fstab	
Extra mount entries for /etc/fstab
– SSH	
SSH daemon config (port, root login, authorized keys)

– Edit
Arbitrary config files to write anywhere on the system

The example configuration file at /etc/zerene/definition.lua explains each section individually so you can easily declare Zerene OS, consider reading it on your installed system.

example config :

return {
  -- SYSTEM IDENTITY
  hostname = "my-workstation",
  timezone = "America/New_York",
  locale = "en_US.UTF-8",
  hwclock = "UTC",
  keymap = "us",
  console_font = "latarcyrheb-sun16",

  -- BOOT
  kernel_cmdline = "mt7925e.disable_aspm=1 pcie_aspm=off",
  boot = {
    timeout      = 5,
    wallpaper    = "boot():/boot/image.png",
    serial       = false,
    serial_speed = 115200,Plan·
  },
/home/gretagen
  -- INIT SYSTEM
  init = "openrc",

  -- ENVIRONMENT VARIABLES
  environment = {
    EDITOR  = "nvim",
    BROWSER = "firefox",
    TERM    = "xterm-256color",
  },

  -- USERS
  users = {
    {
      username = "alice",
      groups   = { "wheel", "audio", "video", "tty", "input", "network" },
      shell    = "/bin/bash",
      ssh_keys = { "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... alice@laptop" },
    },
  },

  -- PACKAGES (managed by zeta)
  packages = {
    "firefox",
    "neovim",
    "alacritty",
    "pipewire",
    "wireplumber",
    "grim",
    "slurp",
    "waybar",
    "hyprland",
    "hyprpaper",
    "hyprlock",
    "wofi",
    "dunst",
    "zathura",
    "mpv",
    "git",
    "gcc",
    "meson",
    "ninja",
  },

  -- SUBSPACES
  subspaces = {
    "debian",
    "arch",
  },

  -- KERNEL MODULES
  modules = {
    "v4l2loopback",
    "i2c-dev",
  },

  -- SERVICES
  services = {
    "sshd",
    "NetworkManager",
    "elogind",
    "pipewire",
    "wireplumber",
  },

  -- NETWORKING
  network = {
    connections = {
      {
        name     = "home-wifi",
        type     = "wifi",
        ssid     = "MyHomeNetwork",
        password = "correct-horse-battery-staple",
        ipv4     = "dhcp",
      },
      {
        name     = "office",
        type     = "ethernet",
        ipv4     = "static",
        address  = "192.168.1.50/24",
        gateway  = "192.168.1.1",
        dns      = { "1.1.1.1", "8.8.8.8" },
      },
    },
  },

  -- FILESYSTEM TABLE
  fstab = {
    extra_mounts = {
      {
        device      = "UUID=1234-ABCD",
        mountpoint  = "/mnt/data",
        fstype      = "ext4",
        options     = "defaults,nosuid,nodev,noatime",
        dump        = 0,
        pass        = 2,
      },
    },
  },

  -- SSH DAEMON
  ssh = {
    enable            = true,
    permit_root_login = false,
    port              = 22,
    authorized_keys   = {
      "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... greta@laptop",
    },
  },

  -- ARBITRARY FILES
  edit = {
    ["/etc/zeta/make.conf"] = [[
BASE_URL="https://mirror.zereneos.org/"
CFLAGS="-O2 -pipe -march=native"
MAKEOPTS="-j8"
]],

    ["/etc/X11/xorg.conf.d/00-keyboard.conf"] = [[
Section "InputClass"
  Identifier "keyboard-all"
  Driver "evdev"
  MatchIsKeyboard "on"
  Option "XkbLayout" "us"
  Option "XkbOptions" "ctrl:nocaps"
EndSection
]],
  },
}


# SUBSPACES

<img width="752" height="422" alt="image" src="https://github.com/user-attachments/assets/c8c6b316-d5f8-48db-9bd0-8828ebf84d6d" />

Subspaces (or subsystems) allow you to use other distributions, such as arch, debian, opensuse into your Zerene OS system without actually installing them.

Think of them as containers that share your kernel but have their own isolated root filesystem.

You can manage subspaces with subspace-cli or subspace-tui
You can enter a subspace with subspace-enter
You can run commands directly inside a subspace without entering it using subspace-run
You can sync subspaces with subspace-sync
You can merge them with subspace-merge
(merging allows you to merge installed packages from a specific or all subspaces into the main system)

When you run an app from a subspace (e.g., subspace-run debian /usr/bin/firefox), Zerene :
- Mounts the subspace's root as /
- Forwards your X11/Wayland display (so GUI windows appear on your desktop)
- Forwards GPU drivers (/dev/dri)
- Forwards D-Bus (for desktop integration)
- Forwards network config (/etc/resolv.conf)
- Isolates IPC, PID, and UTS namespaces
The app thinks it's running on Debian, but it's actually running on your Zerene system with full GPU acceleration.

Available subspaces :

– Debian
– Opensuse
– Alpine
– Fedora
– Gentoo
– Void
– Arch

# INIT SWAPPING

(THIS IS A VERY NEW FEATURE AND IS BOUND TO IMPERFECTIONS, YOU WILL HAVE TO FIX STUFF THAT DO NOT WORK YOURSELF.)

<img width="752" height="423" alt="image" src="https://github.com/user-attachments/assets/9f76b096-9f4e-4066-8070-dd8d2ad55ba7" />

Zerene OS allows you to switch init systems using a command c	allied iniswap, you can also declare your init system in definition.lua

to swap an init system, you must type “initswap [init]”

on next boot, the init system you have chosen to swap to  will be booted. And you will have to make your own services and fix everything that doesn’t seem to be working after the swap yourself, it is recommended to stay on OpenRC.


# end

You now know about every single detail on how Zerene OS works!
If you have any questions or issues; please join the discord below, we hope you enjoy Zerene OS as much as I enjoyed creating it!

<img width="752" height="565" alt="image" src="https://github.com/user-attachments/assets/9fa3a4f2-c4b2-4290-a1f3-772342a1a635" />








