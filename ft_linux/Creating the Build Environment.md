## Escaping the VM

First of all you wont be building this on your host machine. You're gonna want to build this in its own environment. <br>
The way i did this was to use a ~~Gentoo~~  ~~Arch~~ ~~Gentoo~~ [Debian Live CD](http://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/) in a virtual machine. 

Im using Oracel Virtual Machine which I gave 8GB of RAM , 6 cores (dont know how neccessary this is, I think one is ok) and then 25GB of storage space. For someone who knows what they are doing, these choices might not make sense. But I'm learning by doing.

After booting the VM, you will be in a familiar GNOME desktop environment. We can just skipp the installation process and just open a terminal. Then its just a matter of creating a password and enabling ssh.

To change the password, the command is ```passwd```

```sh
sudo passwd user
```

Great, now we need to enable ssh

```sh
# if no sshd
sudo apt-get install ssh

systemctl status sshd  # Show SSH Service status, should be enabled
ip addr show          # Show IP address
```

Now we can immediatley ditch our VM, because it's always better on the Host Machine. We can use ````ssh````to access the Virtual Machine. This way we can easily access documents like the LFS book to assist us in making this project.

```bash
# On Host
ssh user@XXX.XXX.X.X # Double check what the user is called
```

//TODO: FIX

Before we carry on, we need to make sure our host machine is setup to build some of the packages. LFS provides a script to check some packages and aliases

```sh
cat > version-check.sh << "EOF"
#!/bin/bash
# A script to list version numbers of critical development tools

# If you have tools installed in other directories, adjust PATH here AND
# in ~lfs/.bashrc (section 4.4) as well.

LC_ALL=C 
PATH=/usr/bin:/bin

bail() { echo "FATAL: $1"; exit 1; }
grep --version > /dev/null 2> /dev/null || bail "grep does not work"
sed '' /dev/null || bail "sed does not work"
sort   /dev/null || bail "sort does not work"

ver_check()
{
   if ! type -p $2 &>/dev/null
   then 
     echo "ERROR: Cannot find $2 ($1)"; return 1; 
   fi
   v=$($2 --version 2>&1 | grep -E -o '[0-9]+\.[0-9\.]+[a-z]*' | head -n1)
   if printf '%s\n' $3 $v | sort --version-sort --check &>/dev/null
   then 
     printf "OK:    %-9s %-6s >= $3\n" "$1" "$v"; return 0;
   else 
     printf "ERROR: %-9s is TOO OLD ($3 or later required)\n" "$1"; 
     return 1; 
   fi
}

ver_kernel()
{
   kver=$(uname -r | grep -E -o '^[0-9\.]+')
   if printf '%s\n' $1 $kver | sort --version-sort --check &>/dev/null
   then 
     printf "OK:    Linux Kernel $kver >= $1\n"; return 0;
   else 
     printf "ERROR: Linux Kernel ($kver) is TOO OLD ($1 or later required)\n" "$kver"; 
     return 1; 
   fi
}

# Coreutils first because --version-sort needs Coreutils >= 7.0
ver_check Coreutils      sort     8.1 || bail "Coreutils too old, stop"
ver_check Bash           bash     3.2
ver_check Binutils       ld       2.13.1
ver_check Bison          bison    2.7
ver_check Diffutils      diff     2.8.1
ver_check Findutils      find     4.2.31
ver_check Gawk           gawk     4.0.1
ver_check GCC            gcc      5.4
ver_check "GCC (C++)"    g++      5.4
ver_check Grep           grep     2.5.1a
ver_check Gzip           gzip     1.3.12
ver_check M4             m4       1.4.10
ver_check Make           make     4.0
ver_check Patch          patch    2.5.4
ver_check Perl           perl     5.8.8
ver_check Python         python3  3.4
ver_check Sed            sed      4.1.5
ver_check Tar            tar      1.22
ver_check Texinfo        texi2any 5.0
ver_check Xz             xz       5.0.0
ver_kernel 5.4

if mount | grep -q 'devpts on /dev/pts' && [ -e /dev/ptmx ]
then echo "OK:    Linux Kernel supports UNIX 98 PTY";
else echo "ERROR: Linux Kernel does NOT support UNIX 98 PTY"; fi

alias_check() {
   if $1 --version 2>&1 | grep -qi $2
   then printf "OK:    %-4s is $2\n" "$1";
   else printf "ERROR: %-4s is NOT $2\n" "$1"; fi
}
echo "Aliases:"
alias_check awk GNU
alias_check yacc Bison
alias_check sh Bash

echo "Compiler check:"
if printf "int main(){}" | g++ -x c++ -
then echo "OK:    g++ works";
else echo "ERROR: g++ does NOT work"; fi
rm -f a.out

if [ "$(nproc)" = "" ]; then
   echo "ERROR: nproc is not available or it produces empty output"
else
   echo "OK: nproc reports $(nproc) logical cores are available"
fi
EOF

bash version-check.sh
```

Make a note of what is not there. for me its:
- bison
- gawk
- m4
- texinfo
- Under ```alias``` sh is not bash

So we use 

```sh
sudo apt-get install bison gawk m5 texinfo # Plus wget
```

Then to fix the alias of ```sh```:
```sh
sudo ln -sf bash /bin/sh

# Verify change
ls -l /bin/sh # Should see /bin/sh -> bash
```

We can now check our host machine with our ```version-check.sh```:

```bash
user@debian:~$ sh version-check.sh 
OK:    Coreutils 9.7    >= 8.1
OK:    Bash      5.2.37 >= 3.2
OK:    Binutils  2.44   >= 2.13.1
OK:    Bison     3.8.2  >= 2.7
OK:    Diffutils 3.10   >= 2.8.1
OK:    Findutils 4.10.0 >= 4.2.31
OK:    Gawk      5.2.1  >= 4.0.1
OK:    GCC       14.2.0 >= 5.4
OK:    GCC (C++) 14.2.0 >= 5.4
OK:    Grep      3.11   >= 2.5.1a
OK:    Gzip      1.13   >= 1.3.12
OK:    M4        1.4.19 >= 1.4.10
OK:    Make      4.4.1  >= 4.0
OK:    Patch     2.8    >= 2.5.4
OK:    Perl      5.40.1 >= 5.8.8
OK:    Python    3.13.5 >= 3.4
OK:    Sed       4.9    >= 4.1.5
OK:    Tar       1.35   >= 1.22
OK:    Texinfo   7.1.1  >= 5.0
OK:    Xz        5.8.1  >= 5.0.0
OK:    Linux Kernel 6.12.57 >= 5.4
OK:    Linux Kernel supports UNIX 98 PTY
Aliases:
OK:    awk  is GNU
OK:    yacc is Bison
OK:    sh   is Bash
Compiler check:
OK:    g++ works
OK: nproc reports 6 logical cores are available
```

