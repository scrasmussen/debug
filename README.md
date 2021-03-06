# Debugging, Profiling, Git, Makefile, Documentation

_____________________
### Contents

* [Debugging](#debugging)
* [Profiling](#profiling)
* [git](#git)
* [Makefile](#makefile)
* [Documentation](#documentation)
* [Coding Tips](#coding)
* [Coding Culture](#culture)
* [Random Things to Make Your Life Better](#random)


_____________________

### Debugging vs Profiling

What's the difference?

**Debugging**: identify and remove errors from software

**Profiling**: program analysis. Measuring things such as
  * time complexity
  * memory use
  * frequency and duration of function calls

_____________________
<a name="debugging"></a>
### Debugging

We will use `gdb`, the GNU Project debugger.
For a concise list of commands checkout a [cheatsheet](https://darkdust.net/files/GDB%20Cheat%20Sheet.pdf)
In the following code we use the short cuts `s` for `step` and `fin` for `finish`.
```
$ gcc -g example.c -o runMe.exe
$ gdb ./runMe.exe
(gdb) break 16
Breakpoint 1 at 0x400232: file example.c, line 16.
(gdb) run
Starting program /root/src/debug/runMe.exe
Enter the number: 3

Breakpoint 1, main () at example.c:16
16   i = 1;
(gdb) s
18   fact(i, &j, num);
(gdb) s
fact (i=1, j=0x7ffffffee21c, num=3) at example.c:7
7    for (i=1; i<num; i++)
(gdb) s
8    *j=*j*i;
(gdb) print i
$1 = 1
(gdb) print j
$2 = (int *) 0x7ffffffee32c
(gdb) print *j
$3 = 0
(gdb) p num
$4 = 3
(gdb) c
Continuing.
The factorial of 4 is 0
[Inferior 1 (process 5462) exited normally]
```

What is the problem?
_____________________
<a name="profiling"></a>
## Profiling
This is an example of how to profile while using a linux system,
[here is](https://docs.microsoft.com/en-us/visualstudio/profiling/beginners-guide-to-performance-profiling?view=vs-2017)
information about profiling using Visual Studios. The example demonstates [`TAU: Tuning and Analysis Utilities`](https://www.cs.uoregon.edu/research/tau/home.php)
but [`gprof`](http://www.network-theory.co.uk/docs/gccintro/gccintro_80.html) is a common profiler that
you can also use.

* Open a terminal, enter
```
$ ssh s999999@crescent.central.cranfield.ac.uk
```
if off campus enter the following command first
```
$ ssh s999999@hpcgate.cranfield.ac.uk
```


* To find available profiling software type `module keyword profiling`
or `ml keyword profiling`. To see the manual for `module` to read what
the command does and other information, type `man module`.

### TAU: Tuning and Analysis Utilities

* Select module from list: `module load tau` or `ml tau`.
Type `ml` or `module list` to check that the modules have loaded correctly.

* Find the configuration of the TAU Makefile, `ls` the directory to find
other configuration options.

```
$ echo $TAU_MAKEFILE
/apps/software/TAU/2.27-foss-2018a/x86_64/lib/Makefile.tau-papi-mpi-pdt
```
* Compile code, add `--help` to see the compiler options.

```
$ tau_cc.sh tau_example.c -o runMe.exe
$ mpirun -np $NUMPROCESSES ./runMe.exe
```

* A `profile.X.Y.Z` file will be created.
To see the results use `pprof` or `paraprof`.
`pprof` shows the results on the command line and `paraprof` shows the results in a GUI.
Use `pprof` to check the number of times a function is called and how much time is spent in it.
Go to directory `tau/profileTauTest` or `tau/profileTauMPI` to run `pprof`.
This was created and ran with `TAU_MAKEFILE=/apps/software/TAU/2.27-foss-2018a/x86_64/lib/Makefile.tau-papi-pdt-openmp-opari`

```
$ cd tau/profileTauTest
$ pprof
Reading Profile files in profile.*

NODE 0;CONTEXT 0;THREAD 0:
---------------------------------------------------------------------------------------
%Time    Exclusive    Inclusive       #Call      #Subrs  Inclusive Name
              msec   total msec                          usec/call
---------------------------------------------------------------------------------------
100.0        0.061          292           1           1     292312 .TAU application
100.0            2          292           1       20002     292251 TAU_TEST
 98.5          288          288       10001           0         29 A
  0.5            1            1       10001           0          0 B
```

or look at an MPI profile that was compiled and ran with `TAU_MAKEFILE=/apps/software/TAU/2.27-foss-2018a/x86_64/lib/Makefile.tau-papi-mpi-pdt`.
This was run with the command `mpirun -np 2 ./a.out`

```
$ cd tau/profileTauMPI
$ pprof
Reading Profile files in profile.*

NODE 0;CONTEXT 0;THREAD 0:
---------------------------------------------------------------------------------------
%Time    Exclusive    Inclusive       #Call      #Subrs  Inclusive Name
              msec   total msec                          usec/call
---------------------------------------------------------------------------------------
100.0        0.011           81           1           1      81329 .TAU application
100.0        0.129           81           1           4      81318 TEST_MPI
 94.3           76           76           1           0      76721 MPI_Init()
  5.5            4            4           1           0       4467 MPI_Finalize()
  0.0        0.001        0.001           1           0          1 MPI_Comm_size()
  0.0            0            0           1           0          0 MPI_Comm_rank()

NODE 1;CONTEXT 0;THREAD 0:
---------------------------------------------------------------------------------------
%Time    Exclusive    Inclusive       #Call      #Subrs  Inclusive Name
              msec   total msec                          usec/call
---------------------------------------------------------------------------------------
100.0        0.014           81           1           1      81354 .TAU application
100.0        0.121           81           1           4      81340 TEST_MPI
 94.3           76           76           1           0      76754 MPI_Init()
  5.5            4            4           1           0       4464 MPI_Finalize()
  0.0        0.001        0.001           1           0          1 MPI_Comm_size()
  0.0            0            0           1           0          0 MPI_Comm_rank()

FUNCTION SUMMARY (total):
---------------------------------------------------------------------------------------
%Time    Exclusive    Inclusive       #Call      #Subrs  Inclusive Name
              msec   total msec                          usec/call
---------------------------------------------------------------------------------------
100.0        0.025          162           2           2      81342 .TAU application
100.0         0.25          162           2           8      81329 TEST_MPI
 94.3          153          153           2           0      76738 MPI_Init()
  5.5            8            8           2           0       4466 MPI_Finalize()
  0.0        0.002        0.002           2           0          1 MPI_Comm_size()
  0.0            0            0           2           0          0 MPI_Comm_rank()

FUNCTION SUMMARY (mean):
---------------------------------------------------------------------------------------
%Time    Exclusive    Inclusive       #Call      #Subrs  Inclusive Name
              msec   total msec                          usec/call
---------------------------------------------------------------------------------------
100.0       0.0125           81           1           1      81342 .TAU application
100.0        0.125           81           1           4      81329 TEST_MPI
 94.3           76           76           1           0      76738 MPI_Init()
  5.5            4            4           1           0       4466 MPI_Finalize()
  0.0        0.001        0.001           1           0          1 MPI_Comm_size()
  0.0            0            0           1           0          0 MPI_Comm_rank()

```


### Gprof
There is no need to load `Gprof` using `module` since it is part of the GNU project which is available everywhere.
The first step is to compile a program with the `-pg` flag.

```
$ g++ hello.cpp -pg
```

Then run the program

```
$ ./a.out
```
This produces a gmon.out file. For full output just use `gprof`, for a brief summary use `-b`

```
$ gprof -b
Flat profile:

Each sample counts as 0.01 seconds.
 no time accumulated

  %   cumulative   self              self     total
   time   seconds   seconds    calls  Ts/call  Ts/call  name
     0.00      0.00     0.00        1     0.00     0.00  foo(int)


                        Call graph


granularity: each sample hit covers 4 byte(s) no time propagated

index % time    self  children    called     name
                0.00    0.00       1/1           main [5]
		[7]      0.0    0.00    0.00       1         foo(int) [7]
		-----------------------------------------------


Index by function name

   [7] foo(int)
```

_____________________
<a name="git"></a>
## git

__History__: git, an unpleasant person, was named after the creator, Linus Torvalds.
It was created in 2005 as is the defacto method for distributed revision-control.


Here is a good [tutorial](https://product.hubspot.com/blog/git-and-github-tutorial-for-beginners) to walk you through the basics of git.

The most basic steps: create account at [github](https://github.com), create
a new repository, add a file, commit file.
As you continue to modify the file, commit and push changes to save
them in the repo.
Create branches to test new changes to the codebase before merging them
into the trunk.
You want the trunk to always be working and usually release ready.

Also, checkout the .gitignore file in this repo.
It tells git which file to ignore.
_____________________
<a name="makefile"></a>
## Makefile

__Makefiles will make your life easier!!__

Never type long commands or sequences of commands over and over again.
Use Makefiles, tools like `alias`, and write bashscripts to make your life easy.
Here's a [good introduction](http://www.aktau.be/2013/08/07/a-makefile-for-modern-c-programming-on-unix-like-operating-systems/) to proper Makefiles and how to write them.
For an example look at the Makefile in the repo.

_____________________
<a name="documentation"></a>
## Documentation

**Q:** Why should I document?

**A:** "No matter what you are developing, chances are that some day you
or one of your colleagues will have to revisit it. When that day comes,
you will not remember so vividly what you wrote and why." - [tomerpacific](https://medium.freecodecamp.org/why-documentation-matters-and-why-you-should-include-it-in-your-code-41ef62dd5c2f)


[Doxygen](http://doxygen.nl/) is a documentation generator, a tool for writing software reference documentation.
Here is an example of doxygen code, followed by the output.


### Doxygen Example Code

```C++
/**
 * \class ExampleClass
 *
 * \ingroup PackageName
 * (Note, this needs exactly one \defgroup somewhere)
 *
 * \brief Provide an example
 *
 * This class is meant as an example.  It is not useful by itself
 * rather its usefulness is only a function of how much it helps
 * the reader.  It is in a sense defined by the person who reads it
 * and otherwise does not exist in any real form.
 *
 * \note Attempts at zen rarely work.
 *
 * \author (last to touch it) $Author: bv $
 *
 * \version $Revision: 1.5 $
 *
 * \date $Date: 2005/04/14 14:16:20 $
 *
 * Contact: bv@bnl.gov
 *
 * Created on: Wed Apr 13 18:39:37 2005
 *
 * $Id: doxygen-howto.html,v 1.5 2005/04/14 14:16:20 bv Exp $
 *
 */

#ifndef EXAMPLECLASS_H
#define EXAMPLECLASS_H

class ExampleClass
{

 public:

  /// Create an ExampleClass
  ExampleClass();

  /// Create an ExampleClass with lot's of intial values
  ExampleClass(int a, float b);

  ~ExampleClass();

  /// This method does something
  void DoSomething();

  /** This is a method that does so
   * much that I must write an epic
   * novel just to describe how much
   * it truly does. */
  void DoNothing();

  /** \brief A useful method.
   * \param level an integer setting how useful to be
   * \return Output that is extra useful
   *
   * This method does unbelievably useful things.
   * And returns exceptionally useful results.
   * Use it everyday with good health.
   */
  void* VeryUsefulMethod(bool level);

 private:

  const char* fQuestion; ///< the question
  int fAnswer;           ///< the answer

};                              // end of class ExampleClass

#endif  // EXAMPLECLASS_H
}
```

### Generate Documentation
Run the command
```
$ doxygen doxygen_example.h
```
to produce documentation in `html` and `latex` directories.


### Doxygen Example Output
![alt text](doxygen_example.png)
_____________________
<a name="coding"></a>
## Coding Tips
* __*DIE*: Duplication is Evil__.
  * If you are retyping the same long terminal command over and over,
    stick it in a `bash` script or use `alias`.
  * If you are retyping the same code, stick it in a function.
    If that codes needs to be changed, fixing in one place fixes it in
    every place the function is used.

* __Readability__: you should be able to look at a variable and
immediately know what it should be.
```
a = b * c                // this or
success = pizza * coffee // this?

```
I've been told that good code doesn't need documentation because a
person should read a function name and immediately understand what
it's doing.
Maybe a bit extreme but keep this in mind, you will be a stranger to
your code in a week.
Be kind to future you.

* __Useability__:
  * A function should be as short as possible.
    It should do one thing and do it well.
  * Don't have a single forever long program.
    Split code into different files.
  * Have a clear directory structure with clear file names.
    I like to copy open source projects on how they do these things.

* __Write a [pseudocode](https://kopywritingkourse.com/guides/how-to-write-pseudocode/)
outline before touching the keyboard__.
Should you write a ten page paper without an outline?

* __Follow _A_ coding style__.
Choose one and be consistent.
Every language is unique so you probably want to choose a coding
standard tailored for that language.
  * C
    * [Linux kernel coding style](https://www.kernel.org/doc/html/latest/process/coding-style.html)
    * [GNU Coding Standard](https://www.gnu.org/prep/standards/standards.html#Writing-C)
  * C++
    * [C++ Core Guidelines: Stroustrup & Sutter](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)
    * [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)
  * [PEP 8: Python style guide](https://www.python.org/dev/peps/pep-0008/)
  * [Other Options](https://swalladge.id.au/archives/2018/10/15/alternative-code-styles/)

_____________________
<a name="culture"></a>
## Coding Culture
* __Hacking__
  * [The Hacker Attitude](https://goo.gl/KpRhNn)
  * [The Hacker Ethic](https://en.wikipedia.org/wiki/Hacker_culture#Ethics_and_principles)
  * [The Story of Mel](https://goo.gl/3AGaNx)

* __The neverending war ;)__
  * Use [spaces](https://youtu.be/SsoOG6ZeyUI) over tabs
  * Use [Emacs](https://imgs.xkcd.com/comics/real_programmers.png) over Vim

* __A programmer should have [three virtues](http://threevirtues.com)__
  1. Laziness
  2. Impatience
  3. Hubris

_____________________
<a name="random"></a>
## Random Things to Make Life Better
*  __!! Use Emacs and change Caps to Ctrl !!__ Change caps to ctrl by directly editing
the [registry](https://superuser.com/questions/949385/map-capslock-to-control-in-windows-10)
or using (probably easier) [sharpkeys](https://github.com/randyrants/sharpkeys)

* __!! Install `Vimium`!!__   This allows you to use vimium key bindings to surf the internet.
  No need to use a slow mouse to click while browsing.
  * `Vimium` extension for [Chrome](https://vimium.github.io/)
  * `Vimium-FF` extension for [Firefox](https://addons.mozilla.org/en-US/firefox/addon/vimium-ff/)
  * `Vimari` extension for [Safari](https://github.com/guyht/vimari)
  * `Vimoperate` extension for [Opera](https://addons.opera.com/en/extensions/details/vimoperate/)

* Use [Window Subsystem for Linux](http://wsl-guide.org/en/latest/) to
get all the wonderful things of linux on Windows 10!
I've found that [tmux](https://blogs.msdn.microsoft.com/commandline/2016/06/08/tmux-support-arrives-for-bash-on-ubuntu-on-windows/)
is the a good way to get multiple windows/tabs.
