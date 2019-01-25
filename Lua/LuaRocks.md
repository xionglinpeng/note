



```bash
C:\Users\HP\Downloads\luarocks-3.0.4-win32>install.bat /?

C:\Users\HP\Downloads\luarocks-3.0.4-win32>rem=rem --[[--lua
LuaRocks 3.0.x installer.

Installs LuaRocks.

/P [dir]       Where to install LuaRocks.
               Default is %PROGRAMFILES%\LuaRocks

Configuring the destinations:
/TREE [dir]    Root of the local system tree of installed rocks.
               Default is {BIN}\..\ if {BIN} ends with '\bin'
               otherwise it is {BIN}\systree.
/SCRIPTS [dir] Where to install commandline scripts installed by
               rocks. Default is {TREE}\bin.
/LUAMOD [dir]  Where to install Lua modules installed by rocks.
               Default is {TREE}\share\lua\{LV}.
/CMOD [dir]    Where to install c modules installed by rocks.
               Default is {TREE}\lib\lua\{LV}.
/CONFIG [dir]  Location where the config file should be installed.
               Default is to follow /P option
/SELFCONTAINED Creates a self contained installation in a single
               directory given by /P.
               Sets the /TREE and /CONFIG options to the same
               location as /P. And does not load registry info
               with option /NOREG. The only option NOT self
               contained is the user rock tree, so don't use that
               if you create a self contained installation.

Configuring the Lua interpreter:
/LV [version]  Lua version to use; either 5.1, 5.2, 5.3, or 5.4.
               Default is auto-detected.
/LUA [dir]     Location where Lua is installed - e.g. c:\lua\5.1\
               If not provided, the installer will search the system
               path and some default locations for a valid Lua
               installation.
               This is the base directory, the installer will look
               for subdirectories bin, lib, include. Alternatively
               these can be specified explicitly using the /INC,
               /LIB, and /BIN options.
/INC [dir]     Location of Lua includes - e.g. c:\lua\5.1\include
               If provided overrides sub directory found using /LUA.
/LIB [dir]     Location of Lua libraries (.dll/.lib) - e.g. c:\lua\5.1\lib
               If provided overrides sub directory found using /LUA.
/BIN [dir]     Location of Lua executables - e.g. c:\lua\5.1\bin
               If provided overrides sub directory found using /LUA.
/L             Install LuaRocks' own copy of Lua even if detected,
               this will always be a 5.1 installation.
               (/LUA, /INC, /LIB, /BIN cannot be used with /L)

Compiler configuration:
               By default the installer will try to determine the
               Microsoft toolchain to use. And will automatically use
               a setup command to initialize that toolchain when
               LuaRocks is run. If it cannot find it, it will default
               to the /MSVC switch.
/MSVC          Use MS toolchain, without a setup command (tools must
               be in your path)
/MW            Use mingw as build system (tools must be in your path)

Other options:
/FORCECONFIG   Use a single config location. Do not use the
               LUAROCKS_CONFIG variable or the user's home directory.
               Useful to avoid conflicts when LuaRocks
               is embedded within an application.
/F             Remove installation directory if it already exists.
/NOREG         Do not load registry info to register '.rockspec'
               extension with LuaRocks commands (right-click).
/NOADMIN       The installer requires admin privileges. If not
               available it will elevate a new process. Use this
               switch to prevent elevation, but make sure the
               destination paths are all accessible for the current
               user.
/Q             Do not prompt for confirmation of settings
```



```bash
C:\Users\HP\Downloads\luarocks-3.0.4-win32>install.bat /?

C:\Users\HP\Downloads\luarocks-3.0.4-win32>rem=rem --[[--lua
LuaRocks 3.0.x installer.

Installs LuaRocks.

/P [dir]       安装LuaRocks的位置。默认是%PROGRAMFILES%\LuaRocks。
			   %PROGRAMFILES%一般等于C:\Program Files。

Configuring the destinations:
/TREE [dir]    已安装rocks的本地系统树的root。
               默认是{BIN}\..\ 如果{BIN} ends with '\bin'
               否则就是{BIN}\systree.
/SCRIPTS [dir] 在哪里安装rocks安装的命令行脚本。默认是{TREE}\bin。
             
/LUAMOD [dir]  在哪里安装rocks安装的Lua模块。
               默认是{TREE}\share\lua\{LV}.
/CMOD [dir]    在哪里安装rocks安装的c模块。
               默认是{TREE}\lib\lua\{LV}.
/CONFIG [dir]  配置文件应该安装的位置。
               默认是follow /P选项
/SELFCONTAINED 在/P给出的单个目录中创建一个自包含安装。
               将/TREE和/CONFIG选项设置为与/P相同的位置。 
               不加载注册表信息与选项/NOREG。唯一不自包含的选项是user rock tree，
               所以如果您创建一个自包含安装，就不要使用它。

Configuring the Lua interpreter:
/LV [version]  Lua版本使用; 要么 5.1, 5.2, 5.3, 或者 5.4.
               默认是自动检测。
/LUA [dir]     安装Lua的位置-例如c:\ Lua \5.1\
               如果没有提供，安装程序将搜索系统路径和一些默认位置，寻找有效的Lua安装。
               这是基本目录，安装程序将查找子目录bin, lib, include。
               或者可以使用/INC、/LIB和/BIN选项显式指定这些选项。
/INC [dir]     Lua包含的地址 - e.g. c:\lua\5.1\include
               如果提供，覆盖使用/LUA找到的子目录。
/LIB [dir]     Lua库地址 (.dll/.lib) - e.g. c:\lua\5.1\lib
               如果提供，覆盖使用/LUA找到的子目录。
/BIN [dir]     Lua可执行文件地址 - e.g. c:\lua\5.1\bin
               如果提供，覆盖使用/LUA找到的子目录。
/L             安装LuaRocks自己的Lua副本，即使检测到，也总是5.1安装。
               (/LUA, /INC, /LIB, /BIN cannot be used with /L)

Compiler configuration:
               默认情况下，安装程序将尝试确定要使用的Microsoft工具链。
               当LuaRocks运行时，将自动使用setup命令初始化该工具链。
               如果找不到，它将默认为/MSVC开关。
/MSVC          使用MS工具链，不需要安装命令(工具必须在路径中)
/MW            使用mingw作为构建系统(工具必须在您的路径中)

Other options:
/FORCECONFIG   使用单个配置位置。不要使用LUAROCKS_CONFIG变量或用户的主目录。
		       当LuaRocks嵌入到应用程序中时，避免冲突非常有用。
/F             如果安装目录已经存在，则删除它。
/NOREG         不要使用LuaRocks命令加载注册表信息来注册“.rockspec”扩展(右键单击)。
/NOADMIN       安装程序需要管理员权限。 如果不可用，它将提升一个新流程。 
			   使用此开关可防止提升，但请确保当前用户可访问所有目标路径。
/Q             不提示确认设置
```



```
C:\Users\HP\Downloads\luarocks-3.0.4-win32>install.bat /P D:\developer

C:\Users\HP\Downloads\luarocks-3.0.4-win32>rem=rem --[[--lua
LuaRocks 3.0.x installer.


========================
== Checking system... ==
========================


Need admin privileges, now elevating a new process to continue installing...
Now exiting unprivileged installer

C:\Users\HP\Downloads\luarocks-3.0.4-win32>install.bat /P D:\developer\luarocks

C:\Users\HP\Downloads\luarocks-3.0.4-win32>rem=rem --[[--lua
LuaRocks 3.0.x installer.


========================
== Checking system... ==
========================


Need admin privileges, now elevating a new process to continue installing...
Now exiting unprivileged installer
```





```
LuaRocks 3.0.x installer.


========================
== Checking system... ==
========================


Admin privileges available for installing
Looking for Lua interpreter
    checking C:\Program Files (x86)\Common Files\Oracle\Java\javapath
    checking C:\WINDOWS\system32
    checking C:\WINDOWS
    checking C:\WINDOWS\System32\Wbem
    checking C:\WINDOWS\System32\WindowsPowerShell\v1.0
    checking D:\developer\Git\cmd
    checking D:\developer\Java\jdk1.8.0_181
    checking D:\pargram\Maven\apache-maven-3.5.4
    checking C:\WINDOWS\System32\OpenSSH
    checking D:\pargram\Gradle\gradle-4.10.1
    checking D:\developer\Oracle\VirtualBox
    checking D:\developer\Lua\5.1
       Found lua.exe, testing it...
Interpreter found, now looking for link libraries...
    checking for D:\developer\Lua\5.1\lua5.1.lib
    checking for D:\developer\Lua\5.1\lua51.lib
    checking for D:\developer\Lua\5.1\lua5.1.dll
       Found lua5.1.dll
Link library found, now looking for headers...
    checking for D:\developer\Lua\5.1\include\lua\5.1\lua.h
    checking for D:\developer\Lua\5.1\include\lua51\lua.h
    checking for D:\developer\Lua\5.1\include\lua5.1\lua.h
    checking for D:\developer\Lua\5.1\include\lua.h
       Found lua.h
Headers found, checking runtime to use...
    D:\developer\Lua\5.1\lua.exe uses MSVCR80.DLL as runtime
Runtime check completed.
Looking for Microsoft toolchain matching runtime MSVCR80 and architecture x86
    checking: HKLM\Software\Microsoft\VisualStudio\8.0\Setup\VC
    checking: HKLM\Software\Microsoft\VCExpress\8.0\Setup\VS
    Cannot auto-detect Windows SDK version from MSVCR80

==========================
== System check results ==
==========================

Will configure LuaRocks with the following paths:
LuaRocks        : D:\developer\luarocks
Config file     : D:\developer\luarocks\config-5.1.lua
Rocktree        : D:\developer\Lua\5.1\systree

Lua interpreter : D:\developer\Lua\5.1\lua.exe
    binaries    : D:\developer\Lua\5.1
    libraries   : D:\developer\Lua\5.1
    includes    : D:\developer\Lua\5.1\include
    architecture: x86
    binary link : lua5.1.dll with runtime MSVCR80.dll

Compiler        : Microsoft (make sure it is in your path before using LuaRocks)

Press <ENTER> to start installing, or press <CTRL>+<C> to abort. Use install /? for installation options.


============================
== Installing LuaRocks... ==
============================


Installing LuaRocks in D:\developer\luarocks...
Created LuaRocks command: D:\developer\luarocks\luarocks.bat
Created LuaRocks command: D:\developer\luarocks\luarocks-admin.bat

Configuring LuaRocks...
Created LuaRocks hardcoded settings file: D:\developer\luarocks\lua\luarocks\core\hardcoded.lua
Created LuaRocks config file: D:\developer\luarocks\config-5.1.lua

Creating rocktrees...
Created system rocktree    : "D:\developer\Lua\5.1\systree"
Created local user rocktree: "C:\Users\HP\AppData\Roaming\LuaRocks"

Loading registry information for ".rockspec" files

============================
== LuaRocks is installed! ==
============================


You may want to add the following elements to your paths;
Lua interpreter;
  PATH     :   D:\developer\Lua\5.1
  PATHEXT  :   .LUA
LuaRocks;
  PATH     :   D:\developer\luarocks
  LUA_PATH :   D:\developer\luarocks\lua\?.lua;D:\developer\luarocks\lua\?\init.lua
Local user rocktree (Note: %APPDATA% is user dependent);
  PATH     :   %APPDATA%\LuaRocks\bin
  LUA_PATH :   %APPDATA%\LuaRocks\share\lua\5.1\?.lua;%APPDATA%\LuaRocks\share\lua\5.1\?\init.lua
  LUA_CPATH:   %APPDATA%\LuaRocks\lib\lua\5.1\?.dll
System rocktree
  PATH     :   D:\developer\Lua\5.1\systree\bin
  LUA_PATH :   D:\developer\Lua\5.1\systree\share\lua\5.1\?.lua;D:\developer\Lua\5.1\systree\share\lua\5.1\?\init.lua
  LUA_CPATH:   D:\developer\Lua\5.1\systree\lib\lua\5.1\?.dll

Note that the %APPDATA% element in the paths above is user specific and it MUST be replaced by its actual value.
For the current user that value is: C:\Users\HP\AppData\Roaming.


Press any key to close this window...
```

