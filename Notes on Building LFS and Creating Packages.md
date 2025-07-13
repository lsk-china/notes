# Notes on Building LFS and Creating Packages

### 0. PC

​	~~I believe that the most important requirement is a PC with nice CPU.~~

​	Have a good time waiting for compilation and testing of Glibc, GCC, Qt and etc. :-)

### 1. Multilib support

​	Remember to build GCC and so on in Chapter 5 and 6 with multilib support enabled, otherwise you will not be able multilib binarys.

### 2. Build in multi days

​	<b>SET THE LFS ENVIRONMENT VARIABLE IN THE BASHRC OF ROOT USER!!!!</b>

​	<b>DO IT IF YOU DONT WANT TO DESTORY YOUR HOST OS!!!!</b>

### 3. shadow and pam

​	It is OK to build Linux-PAM prior to shadow, but remember to <b>FOLLOW THE INSTRUCTIONS IN BLFS TO BUILD SHADOW</b>, otherwise you'll be unable to set your password.

### 4. Creating software packages

​	You may create a `dest` directory, and specify `DESTDIR` to the `dest` directory to install the content to that directory for packaging.

- When packaging glibc, <b>DO NOT CREATE `dest` DIRECTORY IN THE SOURCE CODE TREE OF GLIBC</b>, otherwise you will receive some errors when `make install`

- Use <b>ABSOLUTE PATH OF THE `dest` DIRECTORY FOR `DESTDIR`</b>, which is required by `make install` of many packages. This can be done by `make DESTDIR=$PWD/dest install` 

    

### 5. Packaging Systemd

​	Use the `DESTDIR` environment variable to let ninja install systemd to your destdir.