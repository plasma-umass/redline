# redline
Redline OS (OSDI 2008)

**<font size="4"><a name="Introduction">Introduction:</a></font>**  
Redline is an operating system kernel designed to ensure responsiveness even when the system is extremely overloaded.  
  **<font size="4"><a name="Downloads">Downloads:</a></font>**  
Currently, Redline works on the <font color="#0000ff">Intel i386</font> platform and supports HyperThreading and SMP, but not NUMA. Please read the instructions carefully when you try out the Redline kernel. While Redline has been tested, you should backup any important things on your machine just in case.

*   CFS-V20.3 patch for kernel 2.6.22.5: [here](http://www.cs.umass.edu/~emery/redline/Code/cfs-2.6.22.5-v20.3.tar.gz)
*   Redline-V1.6 patch against CFS-V20.3: [here](http://www.cs.umass.edu/~emery/redline/Code/redline-v1.6-cfs.tar.gz)
*   Redline-V1.6 patch against kernel 2.6.22.5: [here](http://www.cs.umass.edu/~emery/redline/Code/redline-v1.6-vanilla.tar.gz)
*   SpecTools: [here](http://www.cs.umass.edu/~emery/redline/Code/spectools.tar.gz)
*   Toy programs for testing: [here](http://www.cs.umass.edu/~emery/redline/Code/toys.tar.gz)

**<font size="4"><a name="Instructions">Instructions:</a></font>**  
The Redline kernel uses the Linux kernel 2.6.22.5 as a base, combined with Ingo Molnar's new CFS scheduler which was merged into Linux kernel mainstream summer 2007\. We started our implementation before the merge happened. The Redline kernel adds a new scheduling class that uses EDF with eligibility control in CPU scheduler, modifies virtual memory manager (VMM) to protect pages used by interactive tasks. It also modifies several layers in the system related to I/O management, such as page writeback, the file system, journaling, and the I/O scheduler. All these modifications aim to maintain responsiveness even when the system is overloaded (in terms of CPU, memory or I/O bandwidth).

*   Kernel Installation  
    <font color="#0000ff">Step 1:</font> download Linux kernel 2.6.22.5 [here](http://www.kernel.org), then download CFS V20.3 patch and Redline-V1.6 from [Downloads](index.html#Downloads) section above. Extract kernel source into a local directory, say /<font face="Courier">your_source</font>/, then apply patches using following commands:  

    <font face="Courier">                $cd your_source  
                    $patch -p1 < cfs-v20.3.patch  
                    $patch -p1 < redline-v1.t.patch</font>  
    <font color="#0000ff">Step 2:</font> compile your kernel. Here is an [sample configuration file](http://www.cs.umass.edu/~emery/redline/Code/redline.sample.config) I used.  

    <font face="Courier">                $make xconfig or make menuconfig  
                    $make  
                    $make modules_install install  
    </font>  
    <font color="#ff0000">NOTE:</font> Redline currently only works on Intel i386 platforms. However, Redline implementation does not touch any architecture-specific part, except adding two system calls to support setting and getting specifications for applications. Adding proper system calls into other architectures in the Linux kernel should also make Redline work.  

    <font color="#ff0000">NOTE:</font> You may have to disable the following options in your kernel configuration file, even though by default they are disabled:  
    <font face="Courier">                CONFIG_NUMA        </font>/* For NUMA */<font face="Courier">  
                    CONFIG_MC          </font>/* For Multi-Core */<font face="Courier">CONFIG_RT_MUTEXES</font> /* For Mutex Priority Inheritance */<font face="Courier">  
                    CONFIG_NOMMU       </font>/* no MMU, for systems that have no VM */
*   System Setup using SpecTools  
    Redline is specification-driven, so you need to setup specifications for your system, and also manage them. SpecTools is a package that allows you to do these tasks. It contains a specification table, several binary tools for setting/getting specifications, and several scripts. For details, please refer to README.SPECTOOLS in the downloaded package.  

    <font color="#0000ff">Step 1:</font> Before you can compile SpecTools, you need first add necessary system call interface, because Redline adds two system calls. You need to  
            replace: <font face="Courier">/usr/include/asm/unistd.h </font>  (or whatever suitable file in your system)  
            with    : <font face="Courier">/include/asm-i386/unistd.h</font> (in your downloaded kernel source above)  

    <font color="#0000ff">Step 2:</font> Download SpecTools from the [Downloads](index.html#Downloads) section above, and then decompress it using  

    <font face="Courier">                 $tar -zxvf spectools.tar.gz  
                     $make  
                     $make install</font>       /* You need have root privilege to do this */This will setup a directory <font face="Courier">/etc/spec</font> in your system, and generates proper specification files in that directory from <font face="Courier">specification.tab</font>. You can do this manually using <font face="Courier">update_spec.pl</font> script in the package. It also installs two binaries: <font face="Courier">set_pid_spec</font> and <font face="Courier">get_pid_spec</font>  

    <font color="#0000ff">Step 3:</font> The last thing you need to do is add one line in your <font face="Courier">/etc/rc.d/rc.local</font>  
                             <font face="Courier">/etc/spec/kernelthread_specfix.pl</font>  
    So that your system invokes this script after it boots up. This script will set specifications for kernel threads, which do not call <font face="Courier">do_execve()</font> at all.  

    <font color="#ff0000">NOTE:</font> The sample system specification provided by SpecTools, i.e. <font face="Courier">specification.tab</font>, is configured for <font color="#0000ff">KDE</font> desktop environment. Currently, Redline does not include specifications for GNOME. Most GUI applications currently inherit a default specification from KDE, when <font face="Courier">kdeinit</font> fork and exec a new application. _Firefox_, _mplayer_, _gmplayer_, _vim_, _gvim_ have their own specifications, either because they need more than the default or because they are often launched from a shell. You can provide your own specification for an application if necessary; please see README files in SpecTools package for how to do it.  

    Now, you are ready to reboot your system with Redline kernel. Enjoy!  

*   Setting Specification:  
    Please read README.SPECTOOLS and README.SPECIFICATIONS to see how to provide specification for applications

**<font size="4"><a name="FAQ">FAQ:</a></font>**  
If you have any questions or suggestions, please contact Ting Yang at tingy@cs.umass.edu or Emery Berger at emery@cs.umass.edu.

<small>This material is based upon work supported by the National Science Foundation under CAREER Award CNS-0347339 and CNS-0615211\. Any opinions, findings, and conclusions or recommendations expressed in this material are those of the author(s) and do not necessarily reflect the views of the National Science Foundation.</small>