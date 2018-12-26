Little bit change to run it

2. 解压文件后进入文件根目录并运行以下命令：
  $ autoconf
  $ ./configure
3. 进入lib目录并且make

4. 进入libfree目录并且make
   在该目录运行时出现错误，错误代码如下：

   gcc -I../lib -g -O2 -D_REENTRANT -Wall   -c -o inet_ntop.o inet_ntop.c
   inet_ntop.c: In function ‘inet_ntop’:
   inet_ntop.c:60:9: error: argument ‘size’ doesn’t match prototype
   /usr/include/arpa/inet.h:65:22: error: prototype declaration
   make: *** [inet_ntop.o] Error 1
    经过查询inet_ntop.c 和 inet.h 文件发现在头文件中inet_ntop的原型声明与inet_ntop.c中的该函数实现原型的第三个参数类型不一致

   inet.h 和 inet_ntop.c中的函数原型如下：

   //inet.h
   __const char *inet_ntop (int __af, __const void *__restrict __cp, char *__restrict __buf, socklen_t __len) __THROW;
   //inet_ntop.c
   const char *
   inet_ntop(int af, const void *src, char *dst, size_t size);
   其中第三个参数类型分别为socklen_t和size_t，解决方案为在inet_ntop.c中加入以下代码：

   #define size_t socklen_t


 5. 编译完成后将编译输出的libunp.a拷贝到库文件目录（/usr/lib 和 /usr/lib64）中去:
   sudo cp libunp.a /usr/lib  
   sudo cp libunp.a /usr/lib64 
    
6. 修改头文件unp.h和config.h拷贝到头文件目录中去（/usr/include）
   for MacOS only, need install xcode-select --install or installer -pkg /Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14.pkg -target /   https://apple.stackexchange.com/questions/337940/why-is-usr-include-missing-i-have-xcode-and-command-line-tools-installed-moja 
   gedit lib/unp.h   //将unp.h中#include "../config.h"修改为#include "config.h"
   sudo cp lib/unp.h /usr/include
   sudo cp config.h /usr/include
   
7. 编译测试源代码
 cd ./intro  
 gcc daytimetcpcli.c -o daytimetcpcli -lunp
 gcc daytimetcpsrv.c -o daytimetcpsrv -lunp

QUICK AND DIRTY
===============

Execute the following from the src/ directory:

    ./configure    # try to figure out all implementation differences

    cd lib         # build the basic library that all programs need
    make           # use "gmake" everywhere on BSD/OS systems

    cd ../libfree  # continue building the basic library
    make

    cd ../libroute # only if your system supports 4.4BSD style routing sockets
    make           # only if your system supports 4.4BSD style routing sockets

    cd ../libxti   # only if your system supports XTI
    make           # only if your system supports XTI

    cd ../intro    # build and test a basic client program
    make daytimetcpcli
    ./daytimetcpcli 127.0.0.1

If all that works, you're all set to start compiling individual programs.

Notice that all the source code assumes tabs every 4 columns, not 8.

MORE DETAILS
============

5.  If you need to make any changes to the "unp.h" header, notice that it
    is a hard link in each directory, so you only need to change it once.

6.  Go into the "lib/" directory and type "make".  This builds the library
    "libunp.a" that is required by almost all of the programs.  There may
    be compiler warnings (see NOTES below).  This step is where you'll find
    all of your system's dependencies, and you must just update your cf/
    files from step 1, rerun "config" and do this step again.

6.  Go into the "libfree/" directory and type "make".  This adds to the
    "libunp.a" library.  The files in this directory do not #include
    the "unp.h" header, as people may want to use these functions
    independent of the book's examples.

8.  Once the library is made from steps 5 and 6, you can then go into any
    of the source code directories and make whatever program you are
    interested in.  Note that the horizontal rules at the beginning and
    end of each program listing in the book contain the directory name and
    filename.

    BEWARE: Not all programs in each directory will compile on all systems
    (e.g., the file src/advio/recvfromflags.c will not compile unless your
    system supports the IP_RECVDSTADDR socket option).  Also, not all files
    in each directory are included in the book.  Beware of any files with
    "test" in the filename: they are probably a quick test program that I
    wrote to check something, and may or may not work.

NOTES
-----

- Many systems do not have correct function prototypes for the socket
  functions, and this can cause many warnings during compilation.
  For example, Solaris 2.5 omits the "const" from the 2nd argument
  to connect().  Lots of systems use "int" for the length of socket
  address structures, while Posix.1g specifies "size_t".  Lots of
  systems still have the pointer argument to [sg]etsockopt() as a
  "char *" instead of a "void *", and this also causes warnings.

- SunOS 4.1.x: If you are using Sun's acc compiler, you need to run
  the configure program as

        CC=acc CFLAGS=-w CPPFLAGS=-w ./configure

  Failure to do this results in numerous system headers (<sys/sockio.h>)
  not being found during configuration, causing compile errors later.

- If your system supports IPv6 and you want to run the examples in the
  book using hostnames, you must install the latest BIND release.  You
  can get it from ftp://ftp.vix.com/pub/bind/release.  All you need from
  this release is a resolver library that you should then add to the
  LDLIBS and LDLIBS_THREADS lines.

- IPv6 support is still in its infancy.  There may be differences
  between the IPv6 sockets API specifications and what the vendor
  provides.  This may require hand tweaking, but should get better
  over time.

- If your system supports an older draft of the Posix pthreads standard,
  but configure detects the support of pthreads, you will have to disable
  this by hand.  Digital Unix V3.2C has this problem, for example, as it
  supports draft 4, not the final draft.

  To fix this, remove wrappthread.o from LIB_OBJS in "Make.defines" and
  don't try to build and run any of the threads programs.

COMMON DIFFERENCES
------------------

These are the common differences that I see in various headers that are
not "yet" at the level of Posix.1g or X/Open XNS Issue 5.

- getsockopt() and setsockopt(): 5th argument is not correct type.

- t_bind(): second argument is missing "const".

- t_connect(): second argument is missing "const".

- t_open(): first argument is missing "const".

- t_optmsmg(): second argument is missing "const".

- If your <xti.h> defines the members of the t_opthdr{} as longs,
  instead of t_uscalar_t, some of the printf formats of these value
  might generate warnings from your compiler, since you are printing
  a long without a corresponding long format specifier.
