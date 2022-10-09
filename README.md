============================
*  UPDATE IMGUI FOR UNITY  *
*   you'r gonna love it    *
============================

clone these repos :

 - ImGui.NET   https://github.com/psydack/ImGui.NET.git

 - ImGui.NET-nativebuild (recursively)  https://github.com/psydack/ImGui.NET-nativebuild.git
    . Make sure every sub repository are well cloned. If not, clone them by yourself :
		+ https://github.com/psydack/cimgui.git
			- https://github.com/ocornut/imgui.git
		+ https://github.com/psydack/cimguizmo.git
			- https://github.com/CedricGuillemet/ImGuizmo.git
		+ https://github.com/psydack/cimnodes.git
			- https://github.com/Nelarius/imnodes.git
		+ https://github.com/psydack/cimplot.git
			- https://github.com/epezent/implot.git

we will need to build native c dll. For that, please install :

 - cmake (installed and setup by visual studio once C++ component is enabled on VS, since VS 2017)

 - vcpkg (copy vcpkg folder to C:)  https://github.com/microsoft/vcpkg.git
	+ start vcpkg/bootstrap-vcpkg.bat
 	+ get freetype : "vcpkg install freetype --triplet=x64-windows[-static]", then : "vcpkg integrate install"


===========================================================================

 - Go to "ImGui.NET-nativebuild" folder and run "ci-build.cmd" (handle any cmake error)

	+ if cmake is not reconized as cmd once installed by VS, change the following line into file "ci-build.cmd" : 
		. set PATH="C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin\cmake.exe";%PATH%
		. feel free to change the path according to your visual studio installation
 
	+ if fail to link freetype.lib, copy it from "ImGui.NET-nativebuild/freetypelib" folder to $(WindowsSDK_LibraryPath_x64) folder (C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.33.31629\lib\x64) 


==== * cimgui and friends dll are now generated. be strong, you are far from the end. * ====

Now we need to generate some json files that represent c prototypes for generate .NET code.
For that, we need to install somme stuff :
	- LuaJIT : https://luajit.org/download.html
		+ open vs command prompt, cd to luajit folder/src (cd /d [LUAJIT_folder]\src) and type 'msvcbuild'
	this will generate luajit.exe and lua51.dll into src folder. Copy them to any folder (just to keep them).
	Copy any files from the folder src/jit and past it into the same folder you just copy luajit.exe
	
Now for running .lua scripts, you will need a cpp compiler (yes, we already have msvc, but our .lua can only be generated with gcc).
here is how to install, if this help file miss something : https://www.scaler.com/topics/c/c-compiler-for-windows/
	- Go to https://sourceforge.net/projects/mingw/ and download the installer
	- install and run it
	- check "mingw32-base", "mingw32-gcc-g++", "mingw32-gcc-objc" and click to 'installation' menu and 'Apply changes'
	- once installed, run cmd and type 'g++ --version'
		+ if g++ is not reconized, add MinGW/bin folder path to the PATH environment variable.

before running lua scripts, please go to each  [cimgui/cimguizmo/cimplop/cimnodes]/generator/generator.bat files and edit the line that set PATH variable, to set it with your lua folder (created just above)

You can now run "ImGui.NET-nativebuild\generate-all.cmd". This will call generator.cmd in every cimgui sub repo, that will call generator.lua for each one of them.
Those will generate some json files (check it, into folders [cimgui/cimguizmo/cimplop/cimnodes]/generator/output)

If files are well generated, you can run script "ImGui.NET-nativebuild\copy-jsons-to-generatecode-csharp.cmd"
This will copy json generated files into the ImGUI.NET project at the right place (check it : ImGui.NET\src\CodeGenerator\definitions\[cimgui/cimguizmo/cimplop/cimnodes])


==== * Greate, you just updated files that will be used to generate .NET imgui wrappers * ====

Now we need to run ImGui.NET project, so go into \ImGui.NET\src and open ImGui.NET.sln with VisualStudio
(make sure .NET Core 3 or higher is installed)
Set CodeGenerator project as default startup, switch to release profile and run it
This should have generate some .cs files into ImGui.NET\bin\Release\CodeGenerator\net6.0\gen folder (net6.0 or else, according to your .net core version)

Now edit .bat file : ImGui.NET/CopyGenCodeToSrc.cmd, and adit path for every 'copy' instruction for matching your own relative release path
Once done, run the .bat file
Go to ImGui.NET\src\ImGui.NET\Generated and check if files has been well updated

Now go back to VisualStudio and set ImGui.NET as startup program, and make sure you are on Release profile.
If you look at Dependencies of ImGui.NET and see a litle exclamation warning, don't worry, we will handle it.
	+ expend dependencies and remove UnityEngine (for both .netCore and .netFrameworks)
	+ right click on 'Dependancies" root node and select 'add project reference'
	+ select (left panel) brows, and click on 'brows' button
	+ Go to your unity editor installation and look for UnityEngine.dll (for me it's J:\Editor\2022.1.0b2\Editor\Data\Managed)

You may have to update your local .netstandard version, according to the version the UnityEngine.dll was build.
For that, right click ImGui.NET project and click to properties
Check target frameworks. if netstandard is lower than dll dramework, just go to https://dotnet.microsoft.com/en-us/download/visual-studio-sdks?cid=getdotnetsdk and install same as dll (install dotnet sdk)
Once installed, return to properties on VS and set netstandard to your newly installed netcore version

Now just buil ImGui.NET as release profile
you can build is as Debug but it significantly slow down cimgui native pinvoke
Do the same for each project (ImGui.NET/ImGuizmo.NET/imnodes.NET/ImPlot.NET) => both dll, framework setup and build

You may have to correct some generated code fails (mostly syntaxes on updated imgui native struct).
It's quick and simple, much more than update the code generator.

Once everything has compile, go to => 'ImGui.NET\bin\Release\[ImGui.NET|ImGuizmo.NET|imnodes.NET|ImPlot.NET]' here are your generated ImGui.NET.dll !

now you just have to copy them to your unity imgui plugin folder


good luck
