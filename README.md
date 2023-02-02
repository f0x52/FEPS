# KiCad Project Skeleton

Project skeleton for KiCad projects with KiBot integration using GitHub Actions. Based on a Hackaday article \[1\] and two YouTube video's \[2,3\]: 

- \[1\]: https://hackaday.com/2017/05/18/kicad-best-practises-library-management/
- \[2\]: https://www.youtube.com/watch?v=lODbDxBi9sg
- \[3\]: https://www.youtube.com/watch?v=qJTVHbRei0E

## KiBot GitHub Actions

This skeleton provides three different GitHub workflows to process the KiCad files using KiBot. These workflows look into the [`~/pcb/kicad_project`](pcb/kicad_project) directory for a KiCad project file and process the KiCad schematic and PCB files with the same name. **Because of this, it is not possible to have multiple KiCad project files in this folder without changing the workflow files.**

The workflows execute a string replacement:
- `%%version%%` gets replaces with the git release version number.
- `%%date%%` gets replaces with the date and time on the schematic and the date (without time) in the PCB files.

There are various GitHub workflows present:
- `KiBot - Generate Review Gerbers` - You can run this action manually to generate generic Gerber files for review. 
- `KiBot - Generate Schematic Outputs` - You can run this action manually to gerarate pdf files of schematic and save the BOM as a CSV so you can import it in OctoPart. 
- `KiBot - Release` - Runs when you create a GitHub release and automatically exports Gerber and assembly files for JLCPCB, a PDF file of the schematic, a CSV file of the BOM that you can import in OctoPart, an interactive HTML BOM, and a 3D-model (STEP) of the board.

If you would like export options for other manufacturers, please open an issue or make a pull request. 

## Directory Outline
- [`~/pcb/`](pcb) PCB project.
    - [`~/pcb/.kibot`](pcb/.kibot) KiBot configuration files.
    - [`~/pcb/3d_models`](pcb/3d_models) .STEP and .WRL model files for all project specific footprints.
    - [`~/pcb/datasheets`](pcb/datasheets) Data sheets for components used.
    - [`~/pcb/lib_fp.pretty`](pcb/lib_fp.pretty) Project specific footprint library.
    - [`~/pcb/kicad_project`](pcb/kicad_project) Kicad project. When starting a new KiCAD project in this directory, be sure to uncheck *Create a new directory for the project* in the **Create New Project** window.
    - [`~/pcb/lib_sch`](pcb/lib_sch) Project specific schematic symbol libraries.
        - [`~/pcb/lib_sch/lib.lib`](pcb/lib_sch/lib.lib) Project specific schematic symbol library.
    - [`~/pcb/page_layouts`](pcb/page_layouts) Page layouts for pdf outputs.
    - [`~/pcb/outputs`](pcb/outputs) Project output files generated by KiCad and/or KiBot (these are not kept under version control).
        - [`~/pcb/outputs/3d_models`](pcb/outputs/3d_models) Exported 3D models of the board.
        - [`~/pcb/outputs/assembly`](pcb/outputs/assembly) Exported assembly files.
        - [`~/pcb/outputs/bom`](pcb/outputs/bom) Exported BOM files.
        - [`~/pcb/outputs/fabrication`](pcb/outputs/fabrication) Exported production (gerber) files.
        - [`~/pcb/outputs/images`](pcb/outputs/images) Exported images and 3D board renders.
        - [`~/pcb/outputs/pdf`](pcb/outputs/pdf) Exported schematics, board layouts, dimension drawings.
- [`~/firmware/`](firmware) Firmware souce code and binaries.
- [`~/enclosure/`](enclosure) Mechanical CAD design files for the encloser or box. Create seperate directories for `.stl` or other files for 3d printing for easy accesss.

## Managing Schematic Libraries
Project specific symbols can be saved in [`~/pcb/lib_sch/lib.lib`](pcb/lib_sch/lib.lib). Link this library as a project specific library to the project **using a relative path** like `${KIPRJMOD}/../lib_sch/lib.lib`.

### Saving a copy of the global symbols.
The global symbols are pulled from Github when the repository is cloned, and this can cause problems if the global symbols have changed in the meantime. To prevent this, KiCad automatically creates a backup in the project folder of the symbols currently in use named `<<PROJECT NAME>>-cache.lib`. 

To make sure everyone uses the same, you can convert the backup of the symbol library to a symbol library. To do this you need to move this file to  [`~/pcb/lib_sch`](pcb/lib_sch) and rename it to `<<PROJECT NAME>>.lib`. After this, you can add this symbol library as a project-specific symbol library and replace all symbols with the symbols from this library. **NOTE: Do this only after you've completed your whole schematic design!**

## Managing Footprints
Project specific footprints can be saved in [`~/pcb/lib_fp/lib_fp.pretty`](pcb/lib_fp/lib_fp.pretty). Link this library as a project specific library to the project **using a relative path** like `${KIPRJMOD}/../lib_fp/lib_fp.pretty`. Global symbols that are used can be copied to this folder and replaced with this local variant. 

## Managing 3D models of components
3D models for project specific components should be saved in [`~/pcb/3d_models`](pcb/3d_models). Then they can be linked to the matching footprint **using a relative path** like `${KIPRJMOD}/../3d_models/filename.step`.

## Managing the BOM information
Add the following field name templates in Eeschema (`preferences >> Preferences... >> Field Name Templates`):

| Name     | Visible            | URL                  | _Description_                                                                                     |
|:-------- | :----------------: | :------------------: | :------------------------------------------------------------------------------------------------ |
| `MFN`    | :white_check_mark: | :white_large_square: | _Manufacturer name._                                                                              |
| `MFP`    | :white_check_mark: | :white_large_square: | _Manufacturer product number._                                                                    |
| `LCSC#`  | :white_check_mark: | :white_large_square: | _LCSC/JLCPCB part number (leave part number empty if not using JLCPCB assembly)._                 |
| `Config` | :white_check_mark: | :white_large_square: | _Board configuration to apply this part to or `DNF` if part should not be fitted on final board._ |