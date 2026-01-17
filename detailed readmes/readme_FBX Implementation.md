# TS4 SimRipper FBX Export Implementation - Change Log

## Overview
This document consolidates the full FBX export implementation for TS4 SimRipper, addressing the discontinuation of DAE (Collada) format in Blender 5.x. The FBX exporter creates industry-standard FBX ASCII files with complete mesh geometry, skeleton/armature, skinning weights, textures, and materials that import correctly into Blender 5.x and other 3D applications.

## Issue
- DAE (Collada) format is no longer supported as an import/export option in Blender v5.x

## Summary
A complete FBX ASCII exporter was implemented with the following capabilities:
- Mesh geometry (vertices, normals, UVs)
- Proper coordinate system handling
- Smooth shading support
- Full skeleton/bones (armature)
- Skinning (weights + clusters) with correct bind matrices
- Bind pose
- Texture objects and connections for auto-loading
- Glass/alpha mesh texture routing
- Single "Save As..." UI button with format dropdown

## Files Created

### FBX.cs - Core FBX Exporter
- **Location**: `TS4 SimRipper/src/FBX.cs`
- ~650+ lines of code
- Handles conversion from GEOM to FBX format
- Creates proper FBX document structure with headers, settings, objects, and connections
- Implements skeleton, skinning, clusters, bind pose, texture objects, and diagnostics

### FbxWriter Library (13 C# files)
- **Location**: `TS4 SimRipper/src/FbxWriter/`
- Provides low-level FBX file format reading/writing
- Files include:
  - FbxAsciiReader.cs / FbxAsciiWriter.cs
  - FbxBinaryReader.cs / FbxBinaryWriter.cs
  - FbxDocument.cs / FbxNode.cs / FbxNodeList.cs
  - FbxIO.cs / FbxVersion.cs / FbxException.cs
  - FbxBinary.cs / DeflateWithChecksum.cs / ErrorLevel.cs

## Files Modified

### Enums.cs
- Added `FBX = 5` to `MeshFormat` enum (Line 449)

### Form1.Designer.cs
- Removed multiple export buttons from layout
- Added single **Save As...** button

### Form1.cs
- Wired Save As... button to unified export path
- Added FBX filter constant and event handlers

### PreviewControl.cs
- Added `MeshFormat.FBX` handling to export flow
- Unified Save As flow (single dialog for all formats)
- Ensures texture PNGs (including glass) are written when present

### ColladaDAE.cs
- Material defaults (Phong shininess 0, specular black)
- Glass/wings routing logic

### FbxWriter/FbxAsciiReader.cs
- Added `using System.Globalization;` for compilation
- Fixed float parsing issues (exponent handling, precision)
- Fixed array parsing/length handling issues

## Technical Details

### FBX Document Structure
The exporter creates a complete FBX 7.4 ASCII document:

1. **Header Extension** - FBX version (7400), timestamp, creator info
2. **Global Settings** - Coordinate system (Y-up/Z-up), unit scale (100.0 cm)
3. **Documents Section** - Document metadata, scene hierarchy
4. **Definitions** - Object type definitions with correct counts
5. **Objects**:
   - Geometry Nodes (vertices, normals, UVs, face indices)
   - Model Nodes (mesh model definitions)
   - Material Nodes (Phong materials, specular off)
   - Skeleton/Bones (LimbNode hierarchy)
   - Deformers (Skin + Cluster)
   - Video objects (texture file references)
   - Texture objects (wrapping videos)
   - Bind Pose
6. **Connections** - Links all objects together

### Critical Fixes Applied

#### 1. Matrix Serialization Order (Blender Skinning Fix)
- **Problem**: FBX stores 4x4 matrices in **column-major** order, but `Matrix4D.Values` produces **row-major** arrays
- **Result**: Bind pose/cluster matrices were transposed → mesh "explosion"
- **Fix**: Transpose matrices before writing (`matrix.Transpose().Values`)
- **Applied to**: Cluster Transform, TransformLink, TransformAssociateModel; BindPose matrices

#### 2. Skeleton Local Transforms
- **Problem**: Blender expects coherent relationship between bone locals, globals, and cluster matrices
- **Fix**: Bone locals derived from converted globals enforcing: `parentGlobal * local == global`
- **Result**: No more armature collapse in Rest Position

#### 3. UV Coordinate Handling
- **Problem**: Raw GEOM UVs are upside-down in FBX format
- **Fix**: V-coordinate flipped using `1.0 - v` transformation

#### 4. Smooth Shading
- **Problem**: All edges marked as sharp → faceted appearance
- **Fix**: Normals changed to `IndexToDirect` reference with `NormalsIndex` array; added `LayerElementSmoothing`

#### 5. Scale Correction
- **Problem**: Models imported extremely small (0.01x expected size)
- **Fix**: Unit scale factor set to 100.0 (centimeters)

#### 6. FBX Metadata Correctness
- **Fix**: `Definitions.Count` represents total object count, not type count
- **Fix**: `ObjectType: "GlobalSettings"` included
- **Fix**: Correct deformer counts (Skin + Cluster totals)

#### 7. File Access Conflict
- **Problem**: "The process cannot access the file" error
- **Cause**: Double file stream creation
- **Fix**: `FbxIO.WriteAscii(doc, path)` called directly

### Texture Auto-Loading (Blender)
FBX export emits:
- `Video` objects (file references)
- `Texture` objects (wrapping the video)
- Connections from texture to material properties

This enables Blender to automatically discover and hook up PNGs on import.

### Glass/Alpha Texture Routing
- Glass/alpha meshes preferentially bind to the **glass** texture set when present
- Routing considers shader characteristics (glass/wings), not just filename suffixes
- Fixed bug where glass textures were linked but not written to disk

### Material Defaults (Non-Shiny)
- **DAE**: Phong specular black, shininess 0
- **FBX**: Specular factor 0, specular color black, shininess 0, IOR 1

## Technical Specifications

### FBX Format
- **Version**: FBX 7.4 (7400)
- **Format**: ASCII (human-readable)
- **Encoding**: UTF-8

### Supported Features
| Feature | Status |
|---------|--------|
| Mesh geometry (vertices, faces) | ✅ |
| Vertex normals | ✅ |
| UV coordinates (UV0) | ✅ |
| Smoothing groups | ✅ |
| Basic materials (Phong) | ✅ |
| Coordinate system transformation | ✅ |
| Skeleton/bones | ✅ |
| Skinning/weights | ✅ |
| Bind pose | ✅ |
| Texture auto-loading | ✅ |
| Glass/alpha texture routing | ✅ |
| Multiple UV channels | ❌ |
| Animations | ❌ |
| Morph targets/blend shapes | ❌ |
| Vertex colors | ❌ |

### Dependencies
- **FbxWriter Library**: Integrated as source code
  - Source: https://github.com/hamish-milne/FbxWriter
  - Modified: Added `using System.Globalization;`, fixed parsing issues

### Performance
- Export time: ~1-3 seconds for average Sim
- File size: Comparable to DAE files (~1-5 MB)
- Memory usage: Minimal additional overhead

## References
- [FBX ASCII Format Tutorial](https://banexdevblog.wordpress.com/2014/06/23/a-quick-tutorial-about-the-fbx-ascii-format/)
- [FBX File Format Documentation](https://github.com/assimp/assimp/wiki/FBX-File-format)
- [Blender FBX Binary Format Specification](https://code.blender.org/2013/08/fbx-binary-file-format-specification/)
- [FbxWriter GitHub Repository](https://github.com/hamish-milne/FbxWriter)

### Version 1.2 (January 9, 2026)
- Texture auto-loading parity with DAE
- Alpha/glass mesh texture routing
- Glass textures reliably written to disk
- Blender-friendly material defaults (specular off)
- Single Save As... button with format dropdown
