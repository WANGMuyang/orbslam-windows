# ORB_SLAM2_Windows
Easy build for ORB Slam 2 on Windows from Linux

1. Copy ORB from Linux to Windows

2. Build DBoW
- Delete build and lib folder which was generated when building DBoW2 in Linux
- Make a directory called for Cmake orbslam-windows/Thirdparty/DBoW2/build
- Run CMake GUI and set source code to orbslam-windows/Thirdparty/DBoW2 and where to build the binaries to orbslam-windows/Thirdparty/DBoW2/build
- Press Configure and choose Visual Studio 14 2015 Win64 or Visual Studio 12 2013 Win64
- Press Generate. Error: CMake Error at CMakeLists.txt:32 (message):  OpenCV > 2.4.3 not found.
- Check Cmakelists. Comment find_package and add set.

```
# find_package(OpenCV 3.0 QUIET)
# if(NOT OpenCV_FOUND)
   # find_package(OpenCV 2.4.3 QUIET)
   # if(NOT OpenCV_FOUND)
      # message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
   # endif()
# endif()
set(OpenCV_INCLUDE_DIRS C:/Lib/OpenCV32/opencv/build/include)
set(OpenCV_LIBS C:/Lib/OpenCV32/opencv/build/x64/vc14/lib/opencv_world320.lib)
```

- Open the resulting project in the build directory in Visual Studio
- Change build type to Release (should initially say Debug)
- Project(DBoW2 not ALL_BUILD) -> Properties -> General: change Target Extension to .lib and Configuration Type to Static Library (.lib)
- Go to C/C++ Tab -> Code Generation and change Runtime Library to Multi-threaded (/MT)
- Right click All_BUILD and press Build. You should get lots of warnings.
- One error . Change to stdint
```
1>D:\Code\Visual-SLAM\Thirdparty\DBoW2\DBoW2\FORB.cpp(16): fatal error C1083: Cannot open include file: 'stdint-gcc.h': No such file or directory
```
-These result in a lib file.

2. Make a directory called build in orbslam-windows/Thirdparty/g2o
- Use the lib file in this repo directly or follow
- Run CMake GUI and set source code to orbslam-windows/Thirdparty/g2o and where to build the binaries to orbslam-windows/Thirdparty/g2o/build
- Press Configure and choose Visual Studio 14 2015 Win64 or Visual Studio 12 2013 Win64
- Press Generate
- Open the resulting project in the build directory in Visual Studio
- Change build type to Release (in white box up top, should initially say Debug)
- Right click on g2o project -> Properties -> General: change Target Extension to .lib and Configuration Type to Static Library (.lib)
- Go to C/C++ Tab -> Code Generation and change Runtime Library to Multi-threaded (/MT)
- Go to C/C++ -> Preprocessor and press the dropdown arrow in the Preprocessor Definitions, then add a new line with WINDOWS on it (no underscore), then press OK, then Apply
- Build ALL_BUILD.

3. Make a directory called build in orbslam-windows/Thirdparty/Pangolin
- Use the lib file in this repo directly or follow
- Run CMake GUI and set source code to orbslam-windows/Thirdparty/Pangolin and where to build the binaries to orbslam-windows/Thirdparty/Pangolin/build
- Press Configure and choose Visual Studio 14 2015 Win64 or Visual Studio 12 2013 Win64. You'll have a lot of RED and a lot of things that say DIR-NOTFOUND but as long as the window at the bottom says Configuring Done you're fine
- Press Generate
- Open the resulting project in the build directory in Visual Studio
- Change build type to Release (in white box up top, should initially say Debug)
- Build ALL_BUILD. You'll have an error by project testlog that says "cannot open input file 'pthread.lib'" but that doesn't matter cause we don't use testlog. Everything else should build fine, i.e., you should have
========== Build: 18 succeeded, 1 failed, 0 up-to-date, 0 skipped ==========
Error occured when building Pangolin for glew. Use gitbash to do 
```
git clone https://github.com/stevenlovegrove/Pangolin.git
cd Pangolin
mkdir build
cd build
cmake ..
cmake --build .
```

4. Make a directory called build in orbslam-windows
- Cmakeists: Build type: Debug, Release, RelWithDebInfo and MinSizeRel
- Run CMake GUI and set source code to orbslam-windows and where to build the binaries to orbslam-windows/build
- Press Configure and choose Visual Studio 14 2015 Win64 or Visual Studio 12 2013 Win64
- Press Generate
#- Open the resulting project in the build directory in Visual Studio
#- Change build type to Release (in white box up top, should initially say Debug)
#- Right click on ORB_SLAM2 project -> Properties -> General: change Target Extension to .lib and Configuration Type to Static Library (.lib)
#- Go to C/C++ Tab -> Code Generation and change Runtime Library to Multi-threaded (/MT)

5. comment //PANGOLIN_DEPRECATED;
6. #define M_PI 3.14159265358979323846
7. 
- update cmakelist.txt with will call thirdparty lib(g2o and DBoW2) cmakelist.txt so that it will avoid individual build process separately.
8. PANGOLIN_DEPRECATED ,turned out that multiple include directory was linked to platform.h
9. removed usleep with std:thread::sleep_for() 
```
        //usleep(1000);
		std::this_thread::sleep_for(std::chrono::milliseconds(1));
```
10 . error C2039: 'back_inserter': is not a member of 'std'  
add
```
#include <iterator>
```
11. >D:\Code\Visual-SLAM\Thirdparty/DBoW2/DBoW2/TemplatedVocabulary.h(1488): error C2131: expression did not evaluate to a constant
```
  char buf[size_node]; 
  //change to
  char buf = new char[size_node]; 
```

12. >D:\Code\Visual-SLAM\src\MapPoint.cc(273): error C2131: expression did not evaluate to a constant
```
    -const size_t N = vDescriptors.size();		 + size_t N = vDescriptors.size();
							 + std::vector<std::vector<float> > Distances;
     -float Distances[N][N];		                 +Distances.resize(N, vector<float>(N, 0));
     -vector<int> vDists(Distances[i],Distances[i]+N);	 + vector<int> vDists(Distances[i].begin(),Distances[i].end());
```
13. 1>d:\code\visual-slam\thirdparty\g2o\g2o\core\matrix_operations.h(51): fatal error C1001: An internal error has occurred in the compiler.

14. d:\code\visual-slam\thirdparty\g2o\g2o\core\matrix_operations.h(51): fatal error C1001: An internal error has occurred in the compiler.
Change it according to https://github.com/RainerKuemmerle/g2o/issues/91
But Failed
- Copy and replace file from https://github.com/RainerKuemmerle/g2o/blob/master/g2o/core/matrix_operations.h
- VectorX not defined. Found it defined as VectorXd in newest version, change it to vectorXd
-Too many errors. Decided to compile the newest version of G2O.


15. Rebuild g2o following https://github.com/RainerKuemmerle/g2o/wiki/Build-on-Windows-with-MSVC
- Install g2o dependencies: Qt and LibQGLViewer 
- For Qt, install only 5.10
- For LibQGLViewer. http://libqglviewer.com/installWindows.html. Use MinGW (available when you install the OpenSource Qt version): launch the Qt Command Prompt from the Start menu and type:
```
cd \path\to\libQGLViewer-2.7.1\QGLViewer
qmake
mingw32-make
```
- Above command prompt method failed. Try Qt creator
** QtCreator: open the QGLViewer/QGLViewer.pro project file and "build all".  
** Then copy the generated QGLViewer2.dll and QGLViewer2d.dll to a system shared directory such as C:\Windows\System32. 
An alternative is to copy the dll in every executable's directory.



16. Compile g2o after 15.  
- Change cmake path regarding to Qt and LibQViewer
-Use MSBuild Command Prompt for VS2015
```
D:
cd D:\Code\Visual-SLAM\Thirdparty\g2o\build
msbuild g2o.sln /p:Configuration=RelWithDebInfo /maxcpucount
```

17.Recompile my program. (See 4.)
18.  fatal error C1083: Cannot open include file: 'Thirdparty/g2o/g2o/types/types_seven_dof_expmap.h': No such file or directory
-#include"Thirdparty/g2o/g2o/types/sba/types_six_dof_expmap.h"
-#include"Thirdparty/g2o/g2o/types/sim3/types_seven_dof_expmap.h"
19. error C2244: 'g2o::BlockSolver<Traits>::BlockSolver': unable to match function definition to an existing declaration
```
template <typename Traits>
BlockSolver<Traits>::BlockSolver(std::unique_ptr<LinearSolverType> linearSolver)
    :   BlockSolverBase(),
        _linearSolver(std::move(linearSolver))
{
  // workspace
  _xSize=0;
  _numPoses=0;
  _numLandmarks=0;
  _sizePoses=0;
  _sizeLandmarks=0;
  _doSchur=true;
}	
```
20. Cannot solve 19 according to g2o author's reply because we need C++14 VS2017. So use older version of g20 https://github.com/RainerKuemmerle/g2o/releases/tag/20170730_git
21. Redo 16.
- Change some path in EXTERNAL
22. Still cannot solve 19. Use the g2o in ORB_SLAM2
	1>d:\code\visual-slam\thirdparty\g2o\g2o\core\matrix_operations.h(51): fatal error C1001: An internal error has occurred in the compiler.
1>  (compiler file 'msc1.cpp', line 1468)
	INTERNAL COMPILER ERROR in 'C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\x86_amd64\CL.exe'

- Right click on the ORB_SLAM2 project (NOT ALL_BUILD) and click Build
- If you're lucky, that will take few minutes then successfully build!

If you want to build any of the examples (such as mono_euroc), do the following:

- Right click on that project and go to Properties -> C/C++ -> Code Generation, and change Runtime Library to Multi-threaded (/MT). Then press apply
- Right click on it and press build

Then you will find them, say if you do mono_, in (orbslam-windows\Examples\Monocular\Release)
