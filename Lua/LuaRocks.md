



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

