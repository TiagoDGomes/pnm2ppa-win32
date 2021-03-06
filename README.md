Original message: https://sourceforge.net/p/pnm2ppa/mailman/message/268980/

RE: [Pnm2ppa-users] windows solution

From: Andrew van der Stock \<ajv@gr...\> - 2003-06-07 14:16:45

Actually, a long, long time ago, I did a lot of the initial clean up and optimization of the old pbm2ppa code in Visual Studio 6.0 using the C++ compiler. This was before Klamer's excellent ink lookup code was added. The code generally assumes POSIX behavior for the most part. Luckily, this means that even after two and a bit years of major changes, it still builds and works under win32 with a few minor changes. I don't know why you'd want to do so, which is why I never officially added win32 as a target. If you have a lazy Saturday to burn, here's how to build under win32:

Grab a copy of the sources, and have a Visual Studio C++ compiler lying around. Cygwin may just "work", but I haven't tried. I am using Visual Studio.NET 2002 Standard Edition. Untar the sources using WinZip.

Remove the one "inline" in *make_hash_ink.c*. Just use *cl.exe* directly on *make_hash_ink.c* - it's a necessary precursor for building pnm2ppa.

    C:\home\ajv\Visual Studio Projects\pnm2ppa>cl make_hash_ink.c
    Microsoft (R) 32-bit C/C++ Standard Compiler Version 13.00.9466 for 80x86
    Copyright (C) Microsoft Corporation 1984-2001. All rights reserved.
    
    make_hash_ink.c
    Microsoft (R) Incremental Linker Version 7.00.9466
    Copyright (C) Microsoft Corporation.  All rights reserved.
 
    /out:make_hash_ink.exe
    make_hash_ink.obj
 
    C:\home\ajv\Visual Studio Projects\pnm2ppa>make_hash_ink.exe 4 3 > hash_ink.c
    C:\home\ajv\Visual Studio Projects\pnm2ppa>make_hash_ink.exe 1 5 >> hash_ink.c
    C:\home\ajv\Visual Studio Projects\pnm2ppa>dir
    [ ... ]
    07/06/2003  10:46 PM         1,190,590 hash_ink.c

Start a new visual studio project for pnm2ppa, make it an empty Win32 console application. Add all the C files to the project, including the new *hash_ink.c*.

Download a copy of GNU's gengetopt. Untar and grab a copy of *getopt.c*, *getopt1.c* and *gnugetopt.h*. Add all three files to the Visual Studio project.

In *global.h*, add

    #define MAXPATHLEN	1024

just under the `#define` for `VERSION`. This is a furphy - NT supports very very long path lengths, but 1024 is plenty for most users, and realistically Visual Studio's std C library is a bit sucky through extreme neglect, so it's safer to put a short limit on it.

  *  Remove the include for `<sys/param.h>` in *pnm2ppa.c*
  *  Remove the include for `<unistd.h>` in *pnm2ppa.c*
  *  Press control-h to bring up find and replace in all C files...
  *  Replace all 72 instances of `snprintf` with `_snprintf`
  *  Replace all 6 instances of `strcasecmp` with `_stricmp`
  *  Replace all instances of `<getopt.h>` with `"gnugetopt.h"`
  *  Replace all "inline" key words with `"/*` in-line doesn't work `*/"`

The compiler barfs on these as the way pnm2ppa uses inline is a *gcc-ism*. Visual Studio's compiler is *way* smarter at picking these routines than we are anyway, particularly if you optimize for speed.

Choose "Release" build. Right click the project, and choose properties. Open the preprocessor options. Add

    ;__NO_SYSLOG__;HAVE_STRING_H;LANG_EN

to the "Release" project's preprocessor options.

Click "Rebuild". About 2-3 seconds later, you have a working pnm2ppa for win32 in the release directory. There's 26 remaining warnings, some of which point to actual bugs. `size_t` and pointer arithmetic is not done well in pnm2ppa. But for the purposes of this exercise, we will ignore them.


    C:\home\ajv\Visual Studio Projects\pnm2ppa\Release>dir *.exe
    [ ...]
    07/06/2003  11:45 PM           540,672 pnm2ppa.exe
    
    C:\home\ajv\Visual Studio Projects\pnm2ppa>pnm2ppa.exe --verbose -i test.pnm -o test.ppa
    pnm2ppa: Starting print job
    pnm2ppa: Printing  page  1 (PixMap)
    pnm2ppa: Finished rendering page  1
    pnm2ppa: Print job completed successfully.
    
    C:\home\ajv\Visual Studio Projects\pnm2ppa>dir test.ppa
    [ ... ]
 
    07/06/2003  11:50 PM            62,647 test.ppa
 

But the reality is that the HP driver + Ghostscript using GDI or use Ghostscript to make PDF's, and then printing via Acrobat is the best choice.

Thanks,

Andrew
