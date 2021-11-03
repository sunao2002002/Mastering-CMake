[TOC]
# 为啥使用CMake？
If you have ever maintained the build and installation process for a software package, you will be interested in CMake. CMake is an open-source build system generator for software projects that allows developers to specify build parameters in a simple, portable, text file format. This file is then used by CMake to generate project files for native build tools including Integrated Development Environments (IDEs) such as Microsoft Visual Studio or Apple’s Xcode, as well as UNIX, Linux, NMake, and Ninja. CMake handles the difficult aspects of building software such as cross-platform builds, system introspection, and user customized builds, in a simple manner that allows users to easily tailor builds for complex hardware and software systems.

For any project, and especially cross-platform projects, there is a need for a unified build system. Many non CMake-based projects ship with both a UNIX Makefile (or Makefile.in) and a Microsoft Visual Studio workspace. This requires that developers constantly try to keep both build systems up-to-date and consistent with each other. To target additional build systems, such as Xcode, requires even more custom copies of these files, creating an even bigger problem. This problem is compounded if you try to support optional components, such as including JPEG support if libjpeg is available on the system. CMake solves this by consolidating these different operations into one simple, easy-to-understand file format.

If you have multiple developers working on a project, or multiple target platforms, then the software will have to be built on more than one computer. Given the wide range of installed software and custom options that are involved with setting up a modern computer, the chances are that two computers running the same OS will be slightly different. CMake provides many benefits for single platform, multi-machine development environments including:

- The ability to automatically search for programs, libraries, and header files that may be required by the software being built. This includes the ability to consider environment variables and Window’s registry settings when searching.
- The ability to build in a directory tree outside of the source tree. This is a useful feature found on many UNIX platforms; CMake provides this feature on Windows as well. This allows a developer to remove an entire build directory without fear of removing source files.
- The ability to create complex, custom commands for automatically generated files such as Qt’s moc() or SWIG wrapper generators. These commands are used to generate new source files during the build process that are in turn compiled into the software.
- The ability to select optional components at configuration time. For example, several of VTK’s libraries are optional, and CMake provides an easy way for users to select which libraries are built.
- The ability to automatically generate workspaces and projects from a simple text file. This can be very handy for systems that have many programs or test cases, each of which requires a separate project file, typically a tedious manual process to create using an IDE.
- The ability to easily switch between static and shared builds. CMake knows how to create shared libraries and modules on all platforms supported. Complicated platform-specific linker flags are handled, and advanced features like built-in run time search paths for shared libraries are supported on many UNIX systems.
- Automatic generation of file dependencies and support for parallel builds on most platforms.
- 自动生成文件依赖，以及在大多数平台上提供并行构建的支持。

When developing cross-platform software, CMake provides a number of additional features:
在开发跨平台软件时，CMake提供了许多附加的功能：

- 测试机器字节序和其他特定硬件特性的能力。
- 一组编译配置文件可以适用于所有的平台。这避免开发者必须在项目内部以多种不同格式维护相同的信息的问题。
- 支持在它所支持的所有的平台上编译共享库。
- 使用系统相关信息配置文件的能力，例如数据文件的路径和其他信息。CMake可以创建头文件，其中包含以#define 宏形式包含例如数据文件路径以及其他信息。系统特定的标志也可以放在配置头文件中，这比通过命令行-D选型有优势。因为它允许其他构建系统使用CMake构建的库，而无需构建期间使用完全相同的命令行选项。


# CMake的历史
从1999年以来，CMake一直在活跃的开发中，并且已经成熟到成为一系列建问题的解决方案。CMake的开发是作为由美国国家医学图书馆资助的ITK（Insight Toolkit）项目的一部分开始的，ITK是一个大型软件项目，运行在许多平台，且与许多其他软件包进行交互。为了支持这一点，需要一个功能强大且易于使用的构建工具。有在大型项目中使用过构建系统经验的开发者，设计了CMake来满足这些需求。从那时起，Cmake越来越受欢迎，因为它易于使用和灵活，许多项目和开发者都采用它。最有说服力的日子是K桌面环境项目KDE成功使用CMake作为构建系统。KDE可以说是现存的最大的开源软件项目。

CMake还以CTest的形式提供了对软件测试的支持。软件测试过程的一部分包括构建软件，可能需要安装它，以及确定软件的哪一个部分适合当前系统。这使得CTest成为CMake的一个逻辑扩展，因为它已经有了大部分此类信息。

同样的，CMake包含了旨在支持软件跨平台分发的CPack。它提供了一种跨平台的方法为你的软件创建原生安装，利用现有的流行软件包，如Wix，RPM，Cygwin以及PackageMaker。

CMake会在流行的构建工具可用时持续跟踪和支持它们。CMake已迅速为新版本的微软Visual Studio和苹果的Xcode IDE提供支持。此外，CMake还增加了对Google新构建工具Ninja的支持。使用CMake，一旦你编写了输入文件，你就免费获得了对新编译器和构建工具的支持。因为这些支持是内建在CMake的新版本中的，与您的软件发行版本无关。CMake还持续支持交叉编译到其他操作系统或者潜入式设备。CMake的大多数命令在交叉编译时能够正确的处理主机系统和目标平台之间的差异。


# 为啥不用Autoconf？

在开发CMake之前，它的作者有使用一系列现有构建工具的经验。Autoconf和AutoMake的组合提供了一些和CMake相同的功能，但要是在Windows平台上使用这些工具，需要安装许多Windows机器上没有的工具。除了大量工具之外，autoconf难于使用和扩展，也无法实现一些在CMake中很容易的一些任务。即使在你的系统上安装了autoconf和它所需要的环境，它生成的Makefile来强制用户使用命令行。另一方面，CMake提供了一种选择，允许开发者生成可以被Windows和XCode开发人员熟悉的IDE工具直接使用的项目文件。

虽然autoconf支持用户指定的选项，但它不支持依赖选项，即一个选项依赖于另一个属性或选择。例如，在CMake中，您可以根据用户的系统是否支持多线程来决定是否开启多线程选项。CMake提供了一个交互式用户界面，使用户可以轻松查看哪些选项可用以及如何设置它们。

对于Unix用户，CMake叶提供了自动的依赖项生成，这是autoconf无法直接完成的。CMake简单的输入格式也比Makefile.in和configure.in的组合更容易阅读和维护。CMake记忆和链接lib以依赖信息的能力在autoconf/automake中也不存在对等项。

# 为啥不用JAM，qmake，Scons或者ANT？
其它一些工具，如ANT、qmake、SCons和JAM，采取了不同的方法来解决这些问题，它们帮我们塑造了CMake。在这四个工具中，qmake与CMake最相似，尽管它缺乏CMake提供的大部分系统诊断。Qmake的输入格式与传统的Makefile关系更密切。ANT、JAM和SCons也是跨平台的，尽管它们不支持生成本机项目文件。他们确实摆脱了传统的基于Makefile的输入， ANT使用XML，JAM使用自定义的语言，Scons使用Python。其中一些工具直接运行编译器，而不是让系统的构建过程来执行该任务。许多这些工具都需要先安装其他工具如Python或者Java，才能够运行起来。
# 为啥不自己写脚本？
一些项目使用Perl或Python等现有脚本语言来配置构建过程。虽然这样的系统可以实现类似的功能，但过度使用这些工具可能会使构建过程更像寻找复活节彩蛋，而不是一个简单易用的构建系统。在构建软件包时，用户甚至在开始构建过程之前，就被迫找到并安装这个工具的4.3.2以及另一个工具的3.2.4版本。为了避免这个问题，CMake所需的工具不会超过它用于构建的软件所需的工具。使用CMake最小仅需要C编译器、该编译器的本机构建工具和CMake可执行文件。CMake是用C++编写的，只需要一个C++编译器即可构建，而且大多数平台都可以使用预编译的二进制文件。自己编写脚本通常还意味着您无法生成本机Xcode或Visual Studio工作区，从而限制Mac和Windows的构建。

# CMake可以运行在哪些平台？
CMake能够在大多数常见平台上运行，包括微软Windows操作系统，苹果Mac Os X，以及绝大多数Unix或者类Unix平台。同样的，CMake也支持大多数常见编译器。
# CMake有多稳定？

在为项目采用任何新技术或工具之前，开发人员想知道该工具的支持和流行的程度如何。自最初的CMake实现以来，CMake作为一种构建工具越来越受欢迎。开发者和用户社区都在持续增长。CMake在持续开发对新构建技术和工具的支持。
CMake开发团队坚定地致力于向后兼容性。如果CMake曾经可以构建您的项目，它应该始终能够构建您的项目。此外，由于CMake是一个开源项目，源代码始终可供项目根据需要进行编辑和补丁。


