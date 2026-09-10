# Final Fantasy Resonance Character Replacement MOD Tutorial

## Introduction

The overall workflow is similar to the character-replacement MOD workflow used for *Octopath Traveler*. However, FFRS uses **UE5.6**, so some project packaging settings and related tools need to be modified or updated.

After extensive testing and troubleshooting with ChatGPT, the complete workflow is now working. I'm documenting it here as a simple reference for anyone who wants to try it.

---

## Tools & Requirements

### 1. FModel

Used to extract in-game assets.

The DEMO version does not have an AES key available, but you need to generate a `.usmap` file. Here is the `.usmap` for DEMO generated via **UE4SS**, take it if needed.

**[FFRS-Demo.usmap](https://drive.google.com/file/d/1FDl1GnQNb-9ZyMGHG1HwbvI5yTKUdCm2/view)**

### 2. Retoc

FFRS uses the **IoStore** packaging format, so some of the tools used in previous workflows are no longer compatible.

The required command-line instructions will be provided later in this tutorial.

**Download:** <https://github.com/trumank/retoc>

### 3. Unreal Engine 5.6

The packaging settings must have **IoStore** and **Zen Server** enabled. The required settings will be provided below.

---

# Workflow

## 1. Export the Art Assets

Use **FModel** to export the required assets.

- Set the engine version to **UE5.6**.
- Specify the correct `.usmap` file.
- Relevant assets can be found under `Content/Chara`.
- Player character assets are mainly located in:
  - `Field_Unit`
  - `Unit`

## 2. Edit the Assets

Edit the exported sprite sheet as needed.

*This part is omitted from this tutorial.*

## 3. Create a New UE5.6 Project

Create a new **UE5.6** project.

Recreate the directory structure according to the original asset paths, then place the modified sprite sheet into the corresponding directories.

## 4. Modify Texture Settings

The texture settings are basically the same as those used in the *Octopath Traveler* workflow.

Set the following:

- **Mip Gen Settings:** `NoMipmaps`
- **Texture Group:** `2D Pixels`
- **Compression Settings:** `UserInterface2D`
- **Filter:** `Nearest`
- **Never Stream:** ✔

## 5. Configure Project Packaging Settings

Open:

**Project Settings → Packaging**

Modify the following settings:

- **Use Pak File:** ✔
- **Use Io Store:** ✔
- **Use Zen server as cooked output store:** ✔
- **Additional Asset Directories to Cook:** Add a new entry and specify the directory containing the assets that need to be packaged.

  For this example:

  ```text
  /Game/Chara/
  ```

## 6. Create the Cooking Batch File

Create a file named `FinalCook.bat` in the UE project directory.

Paste the following commands into it, save the file, and run it (The main purpose of this `.bat` is actually to use -skipbuild. However, the cooked output still contains a lot of unnecessary files that need to be cleaned up afterward.):

```bat
@echo off
setlocal

set UE_ROOT=F:\UE5\UE_5.6
set PROJECT=F:\UE5\FFRS\FFRS.uproject

echo ========================================
echo UE5.6.1 Cook + IoStore Test
echo ========================================
echo Project:
echo %PROJECT%
echo.

"%UE_ROOT%\Engine\Build\BatchFiles\RunUAT.bat" BuildCookRun ^
-project="%PROJECT%" ^
-platform=Win64 ^
-clientconfig=Development ^
-cook ^
-stage ^
-pak ^
-iostore ^
-skipbuild ^
-nop4 ^
-utf8output

echo.
echo ========================================
echo Finished
echo ErrorLevel=%ERRORLEVEL%
echo ========================================
echo.

pause
```

> **Note:** Modify `UE_ROOT` and `PROJECT` to match your own Unreal Engine and project paths.

## 7. Locate the Cooked/Packed Files

If the process completes without errors, you should find the packaged files in:

```text
<Project Directory>\Saved\StagedBuilds\Windows\FFRS\Content\Paks
```

These files should already use a storage format compatible with the game.

## 8. Extract the Newly Packed Files with Retoc

Open a **CMD** window in the directory where `retoc` is located.

Run:

```text
retoc to-legacy <Project Directory>\Saved\StagedBuilds\Windows\FFRS\Content\Paks <Output Directory>
```

Replace the paths with your actual project and output directories.

## 9. Clean Up the Extracted Files

Go to the output directory.

Delete everything except the `.uasset` / `.uexp` files related to the modified sprites.

## 10. Repack the Cleaned Files with Retoc

Use `retoc` to package the cleaned files back into Zen/IoStore format:

```text
retoc to-zen <Output Directory> <MOD Directory>\MOD_Name_P.utoc --version UE5_6
```

## 11. MOD is Complete

If everything goes correctly, the MOD is now complete.

The final package should contain **three files(.pak/.utoc/.ucas)**.

## 12. Install the MOD

Installation is straightforward.

Copy the three generated MOD files into the game's:

```text
Content\Paks
```

directory.

---

# Summary

The overall workflow is not particularly difficult.

The main issue is that some of the tools that previously supported this workflow do not yet support **UE5.6** (for example, **Zentools**). Therefore, some alternatives had to be found and tested manually.

Fortunately, the complete workflow is now working, providing another practical solution for similar **UE5.6 + IoStore/Zen** MOD projects in the future.
