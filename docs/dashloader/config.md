---
icon: "settings"
---
# Config
## Options
DashLoader has modularized systems which can be toggled in a mods `fabric.mod.json` or in the config by changing the `options` table.

!!! example
    This is how you would disable the option `CACHE_MODEL_LOADER` in your config.
    ```json title="dashloader.json"
    {
      "options": {
        "CACHE_MODEL_LOADER": false
      },
    }
    ```
??? note "Remove from fabric.mod.json"
    This is how you can disable a DashLoader option in your mod. 
    You can use this to easily get DashLoader compatibility but it will also slow down the mod.
    ```json title="fabric.mod.json" 
        "custom": {
            "dashloader:disableoption": [
                "CACHE_MODEL_LOADER"
    		],
        }
    ```

### List of options
| Name        | Impact | Description                          |
| :---------- | : ---- | :----------------------------------- |
| `CACHE_MODEL_LOADER` | :material-chevron-triple-right: Extreme |   Caches BakedModels which allows the game to load extremely fast |
| `CACHE_SPRITES` | :material-chevron-triple-right: Extreme |   Caches Sprite/Atlases which allows the game to load textures extremely fast |
| `CACHE_FONT` | :material-chevron-double-right: High |   Caches all of the fonts and their images. |
| `CACHE_SHADER` | :material-chevron-right: Medium |   Caches the GL Shaders. |
| `FAST_MODEL_IDENTIFIER_EQUALS` | :material-chevron-right: Medium |   Use a much faster .equals() on the ModelIdentifiers |
| `FAST_WALL_BLOCK` | :material-chevron-right: Medium |   Caches the 2 most common blockstates for wall blocks. |
| `CACHE_SPLASH_TEXT` | Small |   Caches the splash texts from the main screen. |

## Compression
The `compression` variable declares what zlib compression level DashLoader should use. This value ranges from 0 to 23. If the `compression` value is 0, no compression will be used.
!!! example
    ```json
    // Default value
    "compression": 3
    ```
    ```json
    // Maximum compression
    "compression": 23
    ```
    ```json
    // No compression
    "compression": 0
    ```
## Splash Lines
The DashLoader config allows for customization of the splash lines at the top of the DashLoader caching toast.

`customSplashLines` is a list of strings which allows you to add custom splash lines to the list where the splash text will be randomly chosen.
!!! example
    ```json
    "customSplashLines": [
        "This documentation was written by notalpha",
        "Typos demolished by glisco"
    ]
    ```

If you want to disable the default splash lines put `addDefaultSplashLines` to `#!c false`
!!! example
    ```json
    "addDefaultSplashLines": false
    ```

