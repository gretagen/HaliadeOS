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
