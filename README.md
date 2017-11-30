GccBuild
========
Truely portable scripts to build Gcc and Gdb on Mac Os X.

Portable ?
-------
Portable means that there is NO requirement for the script to complete. Everything is included. No brew install xxxx !

Works on a fresh os x install. No prerequisite at all.

Why ?
-----
If you want to compile GCC, you google it and it looks simple. Something like download gcc ; execute ./contrib/download_prerequisites ; make a build directory and executre configure, make and make install.

The reality isn't that nice. You start to have a lot of strange errors. Most of the time you google it and then find an answer like : please brew install {some package} before.

If you managed to compile it, your other projects may not compile anymore because of some package that might have replace some command line with the gnu version, or some lib.

I also tried crosstools-ng (sorry crosstools-ng guys) and it fails almost imediately with a cryptic error message. Maybe it was just few missing packages missing.

My script, with the environnment that goes with it, doesn't need anything to be installed. It was test on clean freash install of Mac Os 10.7, 10.8, 10.9, 10.10, 10.11, 10.12, 10.13

How ?
-----
- Clone the repository
- Choose a directory. A sparsebundle will be created there, and only that. So the directory doesn't need to be empty. Let's call it **{parent_dir}**.
- Choose what you want to build. Look in BuildScripts folder, you'll see few files like Build-xxxxxxxxx. I call xxxxxxxxx the **{suffix}**. 
- In terminal, enter : `{path to repository}/Build {parent_dir} {suffix}`. If exists, the sparsebundle is mounted and the build restart from where it was interrupted.
You may have a window asking for installing Xcode command line tools. You can safely click "not now". This window will come back many time.
- You can also, enter : `{path to repository}/CleanAndBuild {parent_dir} {suffix}`. The sparsebundle will be unmounted and renamed by appending -old. A new one is created and build starts.
- You can also, enter : `{path to repository}/Shell {parent_dir} {suffix}`. Then you can use Build or CleanAndBuild (with no arguments), or you can manually enter any command (configure, make, etc.). Useful for creating scripts, see below.

### Environment
Basically, I put all the tools needed to compile in `Tools10.{osx version}` folder.
- Folder `bin_gnu` : one major problem when compiled source developed under Gnu is that Os X has slightly different version of some tools. I put here all the GNU version of command line tools I found (included `ls` !). All theses version comme from **brew**. Thanks to them.
- Folder `bin_apple` : link to few command line tools included in fresh install of os x, needed for compile. If I remeber well, `hdiutil` and `PlistBuddy` were used by scripts to detect sparebundle mount point. But `hdiutil` failed miserably when a sparsebundle is renamed while mounted, that why I wrote some utilities (`is_mountpoint`, `is_sparsebundle_mounted_on`, `umount_sparsebundle` and `sparsebundle_mountpoint`). _Ideally, the gnu version of what's in there should used and nothing should be here_.
- Folder `bin_jief` : few things I wrote to handle sparsebundle mount point, check if a file system is case sensitive and scipts to update path to referenced libs to make things portable.
- Folder `gcc-v4.8.5` : a brew version of gcc 4.8.5 which I use to compile everything. It's possible that a newer gcc willbe needed to compile future release, but so far, it looks like gcc 4.8.5 can compile gcc 7 and gdb 8.
- Folder `AppleDevToolsXXXXXXXXXXX`, `XXXXXXXXXXX` is the version : folder with includes ; apple assembler, linker and few tools ; libs and dynamic lib needed to run. 

Make your own
-------------
Create a Build-yyyyyyyyy and a SetupEnv-yyyyyyyyy in 'BuildScript' directory. You'ready to call `{path to repository}/Build {parent_dir} yyyyyyyyy`

Shell variables :
- `BUILDGCC_STARTUP_DIR` = {parent_dir}
- `BUILDGCC_SUFFIX` = {suffix}
- `BUILDGCC_SOURCENADTOOLS_DIR` = {path to this repository}
- `BUILDGCC_BUILDSCRIPTS` = $BUILDGCC_SOURCENADTOOLS_DIR/BuildScripts. Directory of scripts you don't call directly.
- `BUILDGCC_SOURCESDIR` = $BUILDGCC_SOURCENADTOOLS_DIR/Sources. Directory of sources tar balls.
- `BUILDGCC_PATCHDIR` = $BUILDGCC_SOURCENADTOOLS_DIR/Patch-$BUILDGCC_SUFFIX. Ex : /Volumes/Patch-gcc-494-osx. If you need to store some patch file, it's a good place to do it.
- `BUILDGCC_TOOLS_DIR` = $BUILDGCC_SOURCENADTOOLS_DIR/Tools10.{osx version}
- `BUILDGCC_BUILD_DIR` = /Volumes/Build-$BUILDGCC_SUFFIX. Mount point of the sparsebundle. Root of the build. Ex : /Volumes/Build-gcc-494-osx

#### About Build-xxxxxx scripts
Your build script do as you wants. No obligation, no rules to follow.

If it can help, here's how I structured mines :

Building GCC requires some steps. Each step is a source tar ball to expand, configure and compile. Let's consider the step to compile GMP.

SetupEnv-xxxxxx define `GMP_SOURCES_DIR`, which is the basename of the source tar ball **and** the source diretory after extraction. The build directory will be `$GMP_SOURCES_DIR-build`. It's an out source tree build.

When a step si done sucessfully, a file 'make_sucessfully.done' is created in `$GMP_SOURCES_DIR-build`.

The 1st step is enclosed with something like this :
`if [ ! -f "$BUILDGCC_BUILD_DIR"/$GMP_SOURCES_DIR-build/make_sucessfully.done ]; then`
so a step isn't executed if already done. If the step wasn't successfully done, the content of `$GMP_SOURCES_DIR-build` is deleted and the step is executed.

The others steps are enclosed with, for example : `if [[ -f "$BUILDGCC_BUILD_DIR"/$GMP_SOURCES_DIR-build/make_sucessfully.done   &&  ! -f "$BUILDGCC_BUILD_DIR"/$MPFR_SOURCES_DIR-build/make_sucessfully.done ]]; then`, so the step is executed if previous one is successful and the current isn't.

The step handling is very basic and nothing is automated. Very easy to understand and, in my opinion, way enough.

