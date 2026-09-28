
# Introduction

In film making a `call sheet` is a detailed schedule for arranging casts, scenes, and shots.

For this ComyUI node pack a callsheet is **a prompt of prompts**. `1prompt` to do full production: create characters, voices, scenes, and shots. It can utilize what ever the latests models and nodes that ComfyUI supports, and nearly any preferred custom nodes you want.

Doing everything with one prompt also means a chatbot could produce long form videos just from text alone.

A callsheet document is also easy to use version control with, have you ever tried to diff ComfyUI workflow json files? Insane!

This also opens up opporunties for text modification and generating nodes and models that produce, manage or, intepret callsheet text.

# Status

Currently this is an Alpha release.

The node interfaces may be stable, if they change I may keep older versions around until beta. I will update documents and workflows as changes occur.

# Installation

This node pack is in active development so you have to install it via github, either via command line or download.

command line:
```
cd ComfyUI/custom_nodes
git clone https://github.com/Codes4Fun/callsheet-comfyui.git
```

or [download](https://github.com/Codes4Fun/callsheet-comfyui/archive/refs/heads/main.zip) and open the zip file and copy `callsheet-comfyui-main` to your ComfyUI installed directories `ComyUI/custom_nodes` and rename it to `callsheet-comfyui` (without the main).

Once this is more fully tested and stable it will be available via ComyUI extensions.

# Workflows

I may in the future integrate workflows with more node pack dependencies, for now this is the main one:

[Krea-2 + MiniMax H3 ref2vid](workflows/callsheet_k2_mmh3.json) (depends on KJNodes)

# Support

You can submit [issues on github](https://github.com/Codes4Fun/callsheet-comfyui/issues). You can `@` me in the [ComfyUI Discord dev channel](https://discord.gg/comfyorg).
