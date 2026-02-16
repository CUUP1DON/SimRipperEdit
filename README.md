# TS4 SimRipper Classic
> [!IMPORTANT]
> This program was originally created by [CmarNYC](https://modthesims.info/d/635720/ts4-simripper-classic-rip-sims-from-savegames-v3-14-2-0-updated-4-19-2023.html) & is currently being maintained by [andrews4s](https://github.com/CmarNYC-Tools/TS4SimRipper).

> [!IMPORTANT]
> This program was modified using the help of claude.ai.

> [!WARNING]
> DO NOT create issues about this fork in the original repository! Use the [Issues](https://github.com/CUUP1DON/TS4SimRipper/issues) tab of THIS repository to report bugs or create feature requests.

## Requirements
This program requires [Microsoft .NET Desktop Runtime 10.0](https://dotnet.microsoft.com/en-us/download/dotnet/10.0)

## Download
To download SimRipper, go to [Releases](https://github.com/CUUP1DON/TS4SimRipper/releases) and click the .zip on the latest.

## TS4 Sim Ripper Changelog (2.16.2026)
- Added added CASPART v52 support from the original repo. 

## TS4 Sim Ripper Changelog (1.10.2026)
- Added user friendly error messages
- Prevented UI from sticking when performing an invalid action
- Added FBX export support
- Exchanged DAE, OBJ and MS3D buttons for standard 'Save As...' dialog
- Fixed issue with alpha meshes not being assigned their glass diffuse textures 
- Edited the blender materials it creates so the roughness is at 1 and the specular is at 0, DAE export is unchanged
- Added loading bar and estimated time to remaining to start up
- Added ability to use mouse left click & mouse scroll wheel to move, zoom & pan camera around sim


## TS4 Sim Ripper Changelog (1.8.2026)
- Updated from .NET 6.0 to .NET 10.0
- Ripper will now restart when clicking the save button in the setup dialog to apply and load new directories & package files (should solve the floating heads issue)
- When ripping into collada DAE format, it keeps EA's uvmap naming (uv_0 & uv_1) instead of adding the mesh type (i.e. Top-mesh-map-0, RingMidLeft-mesh-map-1)
- Fixed issue with ripper not extracting simglass shader meshes when using 'All separate meshes, one texture'
- Fixed an issue with ripper overlaying duplicate meshes when exporting sims into the .DAE format
- Included a readme folder for detailed explanations on fixes applied
