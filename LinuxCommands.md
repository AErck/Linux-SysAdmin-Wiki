## Useful Admin Linux Commands and Scripts

### Linux Admin Commands

## Package Info Yum/RPM

To find out where a particular package is installed locally you can use the following:

  rpm -ql <packagename>
  
Typical output for this will look similar to this:
  
    /usr/bin/htop
    /usr/share/doc/htop-2.0.2
    /usr/share/doc/htop-2.0.2/AUTHORS
    /usr/share/doc/htop-2.0.2/COPYING
    /usr/share/doc/htop-2.0.2/ChangeLog
    /usr/share/doc/htop-2.0.2/README
    /usr/share/man/man1/htop.1.gz
    /usr/share/pixmaps/htop.png

This is usefull if you need to specify where an executable is located for your PATH or for configs of other programs such as Spack.
