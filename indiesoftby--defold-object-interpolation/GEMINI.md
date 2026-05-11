## global

> It's a library for the Defold game engine. The Defold engine is a 2D game engine, but it can also be used to make 3D games. It uses Lua 5.1 as its scripting language with "bit" module for bitwise operations.

It's a library for the Defold game engine. The Defold engine is a 2D game engine, but it can also be used to make 3D games. It uses Lua 5.1 as its scripting language with "bit" module for bitwise operations.
Developers write Lua code in the files with `.lua`, `.script`, `.gui_script`, `.render_script`, `.editor_script` extensions. Source code is formatted with 4 spaces for indentation. "snake_case" is used for variable, function, file, folder names. It uses LDoc and EmmyLua annotations for documentation. The example is:

```lua
--- Summary ends with a period.
-- Some description, can be over several lines.
-- @param p1 string First parameter.
-- @param p2 string|nil Third parameter.
-- @return number
-- @see second_fun
function mod1.first_fun(p1,p2)
    -- ...
end
```

Strictly follow the Defold API.

Defold uses Protobuf file format for their metadata files: `.collection`, `.go`, `.sprite`, `.tilemap`, `.tilesource`, `.atlas`, `.font`, `.particlefx,` `.sound`, `.label`, `.gui`, `.model`, `.mesh`, `.material`, `.collisionobject`, `.texture_profiles`, `.display_profiles`, `.appmanifest`, `.manifest`. The `.vp`, `.fp`, `.glsl` files are GLSL shaders. `game.project` is the settings of the project, has the Ini format.

Structure of the project (Note: generated via `tree -L 2 --gitignore --dirsfirst`):
.
├── object_interpolation
│   ├── api - contains Script API docs.
│   ├── commonsrc - Protobuf description for the extension files.
│   ├── editor - IDE helper files.
│   ├── plugins - Built version of the library as .jar file.
│   ├── pluginsrc - Java sources for the build step.
│   ├── src - C++ sources.
├── test
│   ├── 2d - 2D demo project.
├── utils - Helper scripts to build the .jar file
│   ├── build_demo.sh
│   ├── build_plugins.sh
├── game.project - Project settings

---
> Source: [indiesoftby/defold-object-interpolation](https://github.com/indiesoftby/defold-object-interpolation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-06 -->
