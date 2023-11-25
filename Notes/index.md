# Notes
My notes for Chapter 8 of Linux From Scratch - Week 10 of my Operating System Course

#### Scripts:
[Week 10: LFS ch.8](https://raw.githubusercontent.com/KronosDP/os232/master/Notes/ch8.txt) Not done yet. 

# Important
This script still need some testing, please do not run this first (except you want to help me test this, please do help me in testing this script :D)

# Disclaimer
*The sentence on this part is arguably the most important sentence on this webpage. There is no guarantee or any warranty that this will work for the first time. Please (and I really do mean PLEASE) UNDERSTAND EACH AND EVERYTHING THAT YOU TYPE ON YOUR TERMINAL*. You don't want to waste hours of time do you?

# Optimization Problem (optional but very interesting)
*This thing can speed up this week's compilation process*. If you don't want to waste your time, please use more than 4 cores (if your laptop has more than 4 cores). Either ways *skip this part if you don't bother or you have only 4 cores*. 

I don't know, haven't explored it yet but I think we can use 6 cores when compiling this chapter. I'm not really sure tho. The only thing that I'm sure of is that I tried to compile chapter 6 with 6 cores and it speeds it up (my cpu utilization on my task manager (not top on my linux)) increased. The problem is that I don't know if it finished compiling faster. The only thing I know is that when compiling a certain package (I forgot which one is it but I think it's gcc. I'm not sure tho) it takes 1-2 minutes being stuck. Because of that, I stop the compilation process and I think that failed.

Maybe I'm not patient enough, maybe multicore doesn't work. Either ways, try it! It's the only way to find out right? If you want to do 6 cores build of the LFS, go to the setting on your virtual box and go to system. After that, go to processors and increase your core.

Note that I'm having 6 cores on my laptop. If you want to try compiling your linux with more core, remember to set your `MAKEFLAGS` to `-jn` where n is the number of cores you want to use.

Here's how:
```bash
export MAKEFLAGS='-j6'
```


Then check your “LFS”, “ARCH”, “NPROC”, and “MAKEFLAGS” environment variables for root

```bash
echo "LFS=\"$LFS $(df $LFS|tail -1|awk '{print $1,int($2/1000000)"G"}')\" ARCH $(arch) NPROC=$(nproc) MAKEFLAGS=$MAKEFLAGS"
```

output should be (assuming you're using 6 cores) 
```
LFS="/mnt/lfs /dev/sdb2 32G" ARCH x86_64 NPROC=6 MAKEFLAGS=-j6
```

Because chroot is still primitive (translation: we still haven't compile too much thing so yeah, I'll call that primitive) we want to just check NPROC and MAKEFLAGS.

```bash
echo "NPROC=$(nproc) MAKEFLAGS=$MAKEFLAGS"
```

Number of core and makeflags should be the same with what you've set before.


# Setup for Chapter 8
Note that this part is taken from my friend's [github](https://github.com/riorio805/os232/blob/master/NOTES/lfsch8s0-5.md?plain=1). Kudos to him.

## 8.0.S Entering chroot environment
#### Run as `root`
Set `$LFS` variable
```bash
export LFS=/mnt/lfs
```
Check `$LFS` variable (make sure OK)
```bash
echo "===== ======="
echo "Check $LFS..."
if   [ -z $LFS  ] ; then 
  echo ERROR: There is no LFS variable  === ERROR ===
elif [ -d $LFS/ ] ; then
  echo === === === === === === ===  LFS is $LFS/ === OK ===
else
  echo ERROR: There is no LFS directory === ERROR ===
fi
```
Enter chroot environment
```bash
chroot "$LFS" /usr/bin/env -i   \
    HOME=/root                  \
    TERM="$TERM"                \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/usr/bin:/usr/sbin     \
    /bin/bash --login
```

Output should be
```
(lfs chroot) root:/#
```
when you do ls it should be 
```
bin   dev  home  lib64       media  opt   root  sbin     srv  tmp  var
boot  etc  lib   lost+found  mnt    proc  run   sources  sys  usr
```

## 8.0.X Pre-flight Checks
Make sure you are in
`(lfs chroot) root:/# |`
Create `version-check.sh` (from LFS book section 2.2)
```bash
cd /
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

# Coreutils first because-sort needs Coreutils >= 7.0
ver_check Coreutils      sort     7.0 || bail "--version-sort unsupported"
ver_check Bash           bash     3.2
ver_check Binutils       ld       2.13.1
ver_check Bison          bison    2.7
ver_check Diffutils      diff     2.8.1
ver_check Findutils      find     4.2.31
ver_check Gawk           gawk     4.0.1
ver_check GCC            gcc      5.1
ver_check "GCC (C++)"    g++      5.1
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
ver_kernel 4.14

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
EOF
```
Run `version-check.sh` to see version requirements(Make sure all is OK)
```bash
bash version-check.sh
```

## 8.0.B Backup
Leave the chroot environment to get to root\
NOTE: you may need to exit multiple times to get to actual root
```bash
exit
```
Unmount the virtual file system
```bash
mountpoint -q $LFS/dev/shm && umount $LFS/dev/shm
umount $LFS/dev/pts
umount $LFS/{sys,proc,run,dev}
```
Create the backup\
Approximate time required: 6.1 SBU
```bash
cd $LFS
time {
    tar -cJpvf $HOME/lfs-temp-tools-12.0.tar.xz .;
}
cd
```
Mount virtual file systems
```bash
mkdir -pv $LFS/{dev,proc,sys,run}

mount -v --bind /dev $LFS/dev
mount -v --bind /dev/pts $LFS/dev/pts
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run

if [ -h $LFS/dev/shm ]; then
  mkdir -pv $LFS/$(readlink $LFS/dev/shm)
else
  mount -t tmpfs -o nosuid,nodev tmpfs $LFS/dev/shm
fi
```

# Going to Chroot for the Second (or third) Time
After running the things above, you want to go back to `root`.
 
*Please don't forget to do this on `root` to get into `chroot`.*
```bash
chroot "$LFS" /usr/bin/env -i   \
    HOME=/root                  \
    TERM="$TERM"                \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/usr/bin:/usr/sbin     \
    /bin/bash --login
```

Your terminal should look something like this:
```bash
(lfs chroot) root:/#
```

*Important*
If you do `ls -la`, these will show up:
```
bin   dev  home  lib64       media  opt   root  sbin     srv  tmp  var
boot  etc  lib   lost+found  mnt    proc  run   sources  sys  usr
```

**Please make sure that you are on the correct directory** because the bash command below will direct you to the sources diretory.


# All In One Script
Let's be honest, we all want to sleep and let chapter 8 of this LFS build itself while we sleep at night, not knowing if our script works fine or not. With that in mind, I've made a script of all the instructions in chapter 8 of LFS. *Please Keep in mind that I haven't check this Script*. Kindly correct me if there is any mistake on the script because all I do was copy and paste lines of script on LFS chapter 8 (what can I say, I'm only human). Kindly contact me on any sosial media that you find comfortable.

```bash
cd sources
```

Copy paste the script to the terminal (I'm sorry you can't use wget because wget still doesn't exist on our system yet. I've tried it.). *Please confirm again that if you do `ls` you'll see many .tar.xz files*. Here is the [link to the script](https://raw.githubusercontent.com/KronosDP/os232/master/Notes/ch8.txt)

*Before running the script, please pray. Hope that it works and you don't waste 8 hours of sleeping and be greeted with lame errors*

Run the script by simply pressing enter.

Sleep (literally, not your laptop)...

Hours later it should be done...


# Total Time
Well idk, I haven't test the script yet. Againt *I haven't test the script yet*. All I know is just leave it and go to sleep.
