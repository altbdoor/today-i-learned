Cuberite and OpenShift
===

April 13, 2016

I run a small [Minecraft](https://minecraft.net/) instance on [OpenShift](https://www.openshift.com/) for some of my friends (although nobody plays it now) with [Paper](https://github.com/PaperMC/Paper). Since OpenShift have a lot of limitations with CPU and RAM, its challenging to keep it within the set bounds. So, I always look for ways to optimize things, wherever possible.

I found out about [Cuberite](http://cuberite.org/) (formerly MCServer), which was written in C++. Theoratically speaking, C++ should perform better than Java-based Paper. So I set off on a journey to attempt to run Cuberite on OpenShift. From their homepage, they already have a Linux 64-bit build ready for use. Following the guide in the [Manual](https://book.cuberite.org/#1.3), I thought that running `Cuberite` would be a smooth ride. But life is not so simple.

```
./Cuberite: /lib64/libm.so.6: version `GLIBC_2.15' not found (required by ./Cuberite)
./Cuberite: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.20' not found (required by ./Cuberite)
./Cuberite: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.18' not found (required by ./Cuberite)
./Cuberite: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.14' not found (required by ./Cuberite)
./Cuberite: /usr/lib64/libstdc++.so.6: version `CXXABI_1.3.5' not found (required by ./Cuberite)
./Cuberite: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.15' not found (required by ./Cuberite)
./Cuberite: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.19' not found (required by ./Cuberite)
./Cuberite: /lib64/libc.so.6: version `GLIBC_2.14' not found (required by ./Cuberite)
```

It appears that OpenShift has some old version of the libraries, which is why these errors appear. From [AskUbuntu](http://askubuntu.com/questions/421642/libc-so-6-version-glibc-2-14-not-found), it is confirmed that its a library issue, but we do not have any permission to execute `rpm`, so no updating is possible. Fortunately, I was not alone in my quest, as another person had attempted to do the same before this.

From a [Cuberite forum thread](https://forum.cuberite.org/thread-1984.html), a user (Mathias) is attempting to change where Cuberite looks for the libraries. The attempts made were:

- Recompiling Cuberite on OpenShift, not possible, because `gcc` is old.
- Recompiling `gcc`, not possible, will hit file limit quota.
- Changing `$LD_LIBRARY_PATH`, Cuberite does not detect the changes.

Mathias was able to run it successfully at the end with the help of `patchelf`, but the steps were not clearly described. With no other leads, I decided to try this out on my own. First step would be obtaining [PatchELF](http://nixos.org/patchelf.html), where the latest version is 0.9 at time of post. The release did not contain any binaries, so I suppose a build is necessary.

```sh
# we must prefix it into a writable path, because of permission issues
./configure --prefix=~/app-root/data/patchelf

make && make install
```

A `bin` folder should be created in the prefix path, where the binary `patchelf` will reside. Now what is left are the library files. A Google search for `rpm libm.so.6` will reveal a lot of places where one can download them. One particular website which stood out was [RPM PBone Search](http://rpm.pbone.net/) as they list out the library files inside the RPM package.

As a warning, I tried to satisfy the `'GLIBC_2.15' not found` requirement first, but it was later revealed that the `GLIBCXX` would require `GLIBC_2.17` anyways.

Using RPM Pbone, I found a [package](http://rpm.pbone.net/index.php3/stat/4/idpl/26926167/dir/other/com/glibc-2.19-1.ram0.99.x86_64.rpm.html) for `libc.so.6` with `GLIBC_2.19` (well, a later version wouldn't hurt). Since it is a `.rpm` file, we need to extract the contents first. With the help of a tutorial from [nixCraft](http://www.cyberciti.biz/tips/how-to-extract-an-rpm-package-without-installing-it.html), the command to do so is `rpm2cpio package.rpm | cpio -idmv`. Then the hunt is continued for `GLIBCXX` with another [package](http://rpm.pbone.net/index.php3/stat/4/idpl/26926147/dir/other/com/gcc-libstdc++-4.9.1-1.ram0.99.x86_64.rpm.html).

```sh
# again, we need to be in a writable path.
cd ~/app-root/data

mkdir glibc && cd glibc
wget $path_to_glibc_2.19.rpm -O glibc_2.19_package.rpm
rpm2cpio glibc_2.19_package.rpm | cpio -idmv

mkdir ../libstdcpp && cd ../libstdcpp
wget $path_to_libstdcpp_4.9.rpm -O libstdcpp_4.9_package.rpm
rpm2cpio libstdcpp_4.9_package.rpm | cpio -idmv
```

The folder names are just for my understanding. With the libraries obtained, all that is left is using `patchelf` to direct Cuberite to use them. Bear in mind that PatchELF only accepts absolute path. Use `pwd` in the folders to get the absolute paths. From there, its just one long command, as I found out from [Unix StackExchange](http://unix.stackexchange.com/questions/122670/using-alternate-libc-with-ld-linux-so-hacks-cleaner-method).

```sh
$prefix_path=/$absolute_path_to_home/app-root/data

patchelf --set-interpreter $prefix_path/glibc/lib64/ld-linux-x86-64.so.2 \
     --set-rpath $prefix_path/glibc/lib64:/$prefix_path/libstdcpp/usr/lib64 \
     $path_to_cuberite
```

With all that done, Cuberite should be properly patched and can be executed. As a final word, do note that this is not the right way.

> I think it would just be easier to either compile the server yourself on a similar machine and copy it across, or to use a different server. You're having too many problems this way, and It's not really supported. 
> 
> &mdash; bearbin, [Change path where MCServer looks for libraries](https://forum.cuberite.org/thread-1984.html)

I executed Cuberite, and tried to explore a little. However, the chunk generation is pretty bad, and takes a huge hit on the performance. It seems that there is a lot more to be ironed out for Cuberite, but its good to know that a better alternative is in the works.

---

#### References

- https://book.cuberite.org/#1.3
- http://askubuntu.com/questions/421642/libc-so-6-version-glibc-2-14-not-found
- https://forum.cuberite.org/thread-1984.html
- http://nixos.org/patchelf.html
- http://rpm.pbone.net/
- http://unix.stackexchange.com/questions/122670/using-alternate-libc-with-ld-linux-so-hacks-cleaner-method
