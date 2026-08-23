# Zerene OS QnA

# How can I develop for Zerene OS?

 -- You can not be an "active developer" for Zerene OS as it's philosophy contains
 "worked on and made by a single person" however, you can contribute to it in multiple ways.

# How can I contribute to Zerene OS?

-- You can contribute to this distribution in 4 ways, here are the following :

- Writing packages in it's package repository
- Helping, upgrading, or fixing existing components of the OS
- Designing, drawing, or taking pictures for Zerene OS wallpapers
- Post about it or promote it on social media, or introduce people to it

# Is Zerene OS fhs compliant?

-- Yes, Zerene OS is completely file system standard compliant and uses the same tree format as arch or debian 
(with the exceptions of /capture and /subspace)

# Are packages isolated?

-- No, and they will never be isolated , Zerene OS does not use weird path such as "/zerene/store"
for it's packages and adds them to fhs compliant paths instead. ("/usr/bin" for example)

# What filesystem does it support? will there be more?

-- Zerene OS supports multiple filesystem but bootstrapping the OS defaults to btrfs, without it,
the generation feature would not be usable.

# Will Zerene OS support systemd one day?

-- No, it never will, and it is actively discouraged to do any attempts to port systemd to the actual
distro since it would conflict with it's components and go against it's philosophy of being systemd free.
Feel free to fork it's base if you want systemd support yourself, as it will never be implemented to the actual distro.

# Can you use Zerene OS imperatively?

-- Yes, however declaration is recommended to reproduce systems easily.

# Is Zerene OS beginner friendly?

-- No, Zerene OS is not a beginner friendly distro and needs to be manually installed and configured, it requires
intermediate / advanced knowledge in linux systems to set the system up properly, otherwise said,
it is made to be simple to understand and does not require a high learning curve.

# is Zerene OS open source?

-- Yes, and most of it component's source code are also free to view to the public in seperate GitHub repos.

# If you have more questions or issues, feel free to ask them in the discord of the community at https://discord.gg/SyFM6CcHKa
