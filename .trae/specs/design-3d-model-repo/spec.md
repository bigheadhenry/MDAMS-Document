# 3D Model Repository Design Spec (Revised)

## Why
The user wants to expand the "Image Resource Repository Design" (05) to a broader "Resource Repository Design" that includes 3D models. The existing folder `05-image-repo-design` should be renamed to reflect this broader scope, and the new 3D design document should be added there.

## What Changes
- **Rename** directory `prd/05-image-repo-design/` to `prd/05-resource-repo-design/`.
- **Add** new design document `prd/05-resource-repo-design/三维资源库设计（数据层-索引层-服务层-展示层）.md`.

## Impact
- **Affected specs**: `05-image-repo-design` (renamed).
- **Affected code**: None.

## ADDED Requirements
### Requirement: 3D Repository Design Document
The document SHALL cover:
1.  **Data Layer**: 3D formats (Source: OBJ/FBX/PLY vs Delivery: glTF/USDZ), 3D metadata (polygon count, textures, materials).
2.  **Index Layer**: Search capabilities for 3D assets.
3.  **Service Layer**: Processing pipeline for 3D (LOD generation, compression, format conversion).
4.  **Presentation Layer**: Web-based 3D viewers and interaction.

## MODIFIED Requirements
### Requirement: Folder Renaming
- Rename `prd/05-image-repo-design` to `prd/05-resource-repo-design` to accommodate multiple resource types (Image, 3D, Video, etc.).
