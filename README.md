# PlayerUnknown's Battlegrounds | Terrain Maps

PlayerUnknown's Battlegrounds currently features two maps: Erangel and Miramar. This repository provides information and scripts for extracting elevation and normal maps from the game's sources. In addition, the extracted maps are available as well in full, losless detail.

Please note that all preview images are downscaled to 8bit 512px &times; 512px and should not be used for rendering (normal data is downsampled using bicubic resampling).

| Erangel Height Map | Erangel Normal Map |
|--------------------|--------------------|
| <img src="https://github.com/cgcostume/pubg-maps/blob/master/erangel/pubg_erangel_height_l16_preview.png" width="100%" alt="pubg_erangel_elevation_preview"> | <img src="https://github.com/cgcostume/pubg-maps/blob/master/erangel/pubg_erangel_normal_rg8_preview.png" width="100%" alt="pubg_erangel_normal_preview"> |
| Download Height Maps | Download Normal Maps |
| :godmode: [8192px &times; 8192px, 16bit, grayscale, png](https://github.com/cgcostume/pubg-maps/blob/master/erangel/pubg_erangel_height_l16_lod0.png) | :godmode: [8192px &times; 8192px, 8bit, rgb, png](https://github.com/cgcostume/pubg-maps/blob/master/erangel/pubg_erangel_normal_rg8_lod0.png) |

| Miramar Height Map | Miramar Normal Map |
|--------------------|--------------------|
| <img src="https://github.com/cgcostume/pubg-maps/blob/master/miramar/pubg_miramar_height_l16_preview.png" width="100%" alt="pubg_miramar_elevation_preview"> | <img src="https://github.com/cgcostume/pubg-maps/blob/master/miramar/pubg_miramar_normal_rg8_preview.png" width="100%" alt="pubg_erangel_normal_preview"> |
| Download Height Maps | Download Normal Maps |
| :godmode: [8192px &times; 8192px, 16bit, grayscale, png](https://github.com/cgcostume/pubg-maps/blob/master/miramar/pubg_miramar_height_l16_lod0.png) | :godmode: [8192px &times; 8192px, 8bit, rgb, png](https://github.com/cgcostume/pubg-maps/blob/master/miramar/pubg_miramar_normal_rg8_lod0.png) |

## How-To/DIY

Please note that the following steps might change with respect to the PUBG version, asset provisioning and structure.

1. **Download** the UE Viewer by Gildor's Homepage  (`umodel.exe`) - google for it, the sha256 hash of my file is (`56FAB4D29AC7B7800FA6B480D2C2BDA4A7FEC91CCE8B00A8DB77B71849047E96`) and it seems to be legit.
2. **Locate** your PUBG directory, e.g., `C:\Program Files (x86)\Steam\steamapps\common\PUBG`.
3. **Open** `TslGame-WindowsNoEditor_erangel_heightmap.pak` or `TslGame-WindowsNoEditor_desert_heightmap.pak` (or any other of the pak files...). Please note that the pak files were AES encrypted recently (try google the AES key, e.g., on reddit or gildor's forum).
4. **Filter** for `HeightMap` or `Texture2D_` (optional step)
5. **Export** all height maps. This should create a `UmodelExport\Maps\Desert\Art\Heightmap` and `UmodelExport\Maps\Erangel\Art\Heightmap` folder in your current working directory.
6. **Run** `pubg-tga-slice.py` for extracting and encoding the relevant tile data into losless 16bit and 8bit pngs:
```
.\pubg-tga-slice.py -p .\UmodelExport\ -m erangel
.\pubg-tga-slice.py -p .\UmodelExport\ -m miramar
```
That's it. If the script exits without errors there shoud be 8192px &times; 8192px losless height and normal maps.


## How-To/DIY | DEPRECATED (ubulk approach)

Please note that the following steps might change with respect to the PUBG version, asset provisioning and structure.

1. **Download** the UE4 pak-file Unpacker by Haoose v0.5 (`ue4pakunpacker.exe`) - google for it, the sha256 hash of my file is (`A00A579504D0594BE15377DB5DE07D916AD5E1047D15AD555B5A109E71219B5E`) and it seems to be legit.
2. **Locate** your PUBG directory, e.g., `C:\Program Files (x86)\Steam\steamapps\common\PUBG`.
3. **Unpack** `TslGame-WindowsNoEditor_erangel_heightmap.pak` or `TslGame-WindowsNoEditor_desert_heightmap.pak` for Erangel or Miramar respectively. This should create a `TslGame` folder directly in `C:`, i.e., `C:\TslGame\Content\Maps\Erangel\Art\Heightmap` or `C:\TslGame\Content\Maps\Desert\Art\Heightmap` respectively comprising all resources required.

I tried to run steps 1. to 3. via a script as well but couldn't settle on how to provide and handle ue4pakunpacker yet. Feel free to have a look in `pubg-pak-unpack.py`. The following script requires the pip packages: `numpy`, `pypng`, and `Pillow`.

4. **Run** `pubg-ubulk-slice.py` for extracting and encoding the relevant tile data into losless 16bit and 8bit pngs:
```
.\pubg-ubulk-slice.py --map erangel -tsl C:\TslGame --lod 0
.\pubg-ubulk-slice.py --map miramar -tsl C:\TslGame --lod 0
```
That's it. If the script exits without errors there shoud be 8192px &times; 8192px losless height and normal maps. The `--lod` parameter can be used for level of detail of `--lod 0` (8k map), `--lod 1` (4k map), and `--lod 2` (2k map).


#### Details on the Map Encoding

Elevation and normals are packed into 512px &times; 512px &times; 32bit tiles. The first byte and the fourth byte are the 8bit coefficients of the normal. The second and third bytes encode a 16bit elevation/height. Moreover, every `.ubulk` tile file encodes the 512px &times; 512px tile as well as additional downscaled variations (mipmaps), probably LOD1 and LOD2. The binary size of each tile file accumulates to: 512px [width] &times; 512px [height] &times; 4bytes [1byte per channel] &times; (1 [lod0] + 0.25 [lod1] + 0.0625 [lod2]) = 1376256bytes = 1.31MiB

The following paragraphs enumerate the relevant tiles: the first two indices identify the `Heightmap_x#_y#_sharedAssets` group/directory, the following array contains the indices of tiles with normal/elevation data encoded.
