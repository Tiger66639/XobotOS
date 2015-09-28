PREREQUISITES:
==============

(*) Do not attempt to run this on OpenSuSE 12.1.

    If you do have OpenSuSE 12.1, then start Eclipse, create a simple "Hello World"
    in Java, compile and run it - if that works for you without crashing, then you
    may be lucky.
    
    They did something in their glibc that's causing random crashes in Sun's JVM,
    this has made Eclipse so unstable that I had to downgrade to OpenSuSE 11.4
    
(*) Prior to building, you need to increase memory size in Eclipse
    (http://wiki.oseems.com/application/eclipse/fix-gc-overhead-limit-exceeded)

    I'm using -Xms2048m and -Xmx4096m.

(*) To get Fonts working on Linux, you must follow these instructions:
    http://code.google.com/p/chromium/wiki/LinuxOpenSuseBuildInstructions
    (creating all these symlinks is required).

SETTING THINGS UP:
==================

You need to create to separate Eclipse workspaces for this and checkout the
XobotOS module in each of these.  I'm using /work/workspace and /work/test.

If you start Eclipse for the first time, then it'll ask you for the workspace
directory.  Once it created that, checkout XobotOS in it.

In Eclipse, do File / Import / General / Existing Projects into Workspace,
select /work/workspace/XobotOS as root directory, then it should offer you to
import 5 projects.  Select sharpen.core and sharpen.xobotos (sharpen.feature
and sharpen.site are not used at the moment, and you do not want 'android' in
this workspace).  Make sure _not_ to copy them into the workspace, that checkbox
must not be checked.

When imported, you should see the 'sharpen.core' and 'sharpen.xobotos' projects.

Select Project / Build automatically.

Open sharpen.xobotos/plugin.xml.  At the top of the "Overview" tab, you should
see four buttons, the leftmost of these is "Launch an Eclipse application".
Click it, and another instance of Eclipse will start.

The first time this inferior instance of Eclipse starts up, it will ask you for
a workspace directory.  Checkout XobotOS inside that directory, then do the import
as described above, but only select the 'android' module.

You are now ready :-)

You only need to follow the instructions in this section the first time that you're
running this in Eclipse.  Eclipse remembers which projects you had open and your last
run configuration, so when you come back, you can simply start Eclipse, then do
Run / Run (or Run / Debug to debug it).

RUNNING THE BUILDER:
====================

Well, that's easy.

Simply do Project / Build, then get some coffee or take a walk ....

Current time for a full build is about 10 minutes on my machine.

When done, all the generated sources are placed in the output/ directory and
XobotOS-debug.csproj is automatically updated.

BUILDING THE C# SOURCES:
========================

There is a XobotOS-debug.sln file in the top level directory (/work/test/XobotOS/XobotOS-debug.sln),
simply open this in MonoDevelop or use xbuild.

It is highly recommended to close the solution in MonoDevelop before doing a full build.

BUILDING THE C# SOURCES FROM GITHUB:
====================================

You can also directly compile the C# sources from github:

After checking out or updating, simply open XobotOS.sln in MonoDevelop.

Note that there are two solutions:

* XobotOS.sln uses the sources from github.

* XobotOS-debug.sln uses the freshly convereted files from the android/output
  directory.

BUILDING THE SAMPLES:
=====================

This is a little bit complicated and still unstable, but here's how it should work:

First, check http://code.google.com/p/skia/wiki/GettingStartedOnLinux or
http://code.google.com/p/skia/wiki/GettingStartedOnWindows and install all the
prerequisites listed in there.

On Linux, you also need to follow these instructions:
http://code.google.com/p/chromium/wiki/LinuxOpenSuseBuildInstructions.

Create a directory called 'build' if it doesn't already exist, cd into that directory,
then run

  (cd .. && python gyp_xobotos)

This will generate a Makefile in the build/ directory, running 'make' will
build all the native Skia and Android code.

The output gets placed in build/Debug/lib.target.  You either have to add that
directory to your LD_LIBRARY_PATH or manually copy the libxobotos.so.

Open XobotOS.sln or XobotOS-debug.sln in MonoDevelop (depending on whether you
want to use the sources from github or the newly converted ones, see above)
and build the solution.

SAMPLE ARCHITECTURE:
====================

There's a class called XobotOS.Runtime.ActivityManager which is responsible
for running a sample Activity.

We use a modified version of Android's 'aapt' tool called 'xorpt', which compiles
the AndroidManifest.xml and the resources into a '<assembly>-res.zip' and also
creates a R.cs.

TestActivity.csproj has a pre-build rule which automatically runs xorpt:

    <CustomCommands>
      <CustomCommands>
        <Command type="BeforeBuild" command="${TargetDir}/xorpt p -f -S res -F TestActivity-res.zip -J . -M AndroidManifest.xml -I ${SolutionDir}/android/system-root/framework/framework-res.apk" workingdir="${ProjectDir}" pauseExternalConsole="true" />
      </CustomCommands>
    </CustomCommands>

On startup, ActivityManager uses android.content.pm.PackageParser to parse the
resource file and get the startup activity from the AndroidManifest.xml.

We already support constructing a view from XML.
	
