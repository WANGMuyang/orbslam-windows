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
I had to disable warnings in Orb Slam because otherwise there were so many they crashed visual studio. You will still see a few but not very many

- Right click on the ORB_SLAM2 project (NOT ALL_BUILD) and click Build
- If you're lucky, that will take few minutes then successfully build!

If you want to build any of the examples (such as mono_euroc), do the following:

- Right click on that project and go to Properties -> C/C++ -> Code Generation, and change Runtime Library to Multi-threaded (/MT). Then press apply
- Right click on it and press build

Then you will find them, say if you do mono_, in (orbslam-windows\Examples\Monocular\Release)
