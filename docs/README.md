
# Introduction

In film making a `call sheet` is a detailed schedule for arranging casts, scenes, and shots.

For this ComfyUI node pack a callsheet is **a prompt of prompts**. `1prompt` to do full production: create characters, voices, scenes, and shots. It can utilize what ever the latests models and nodes that ComfyUI supports, and nearly any preferred custom nodes you want.

Doing everything with one prompt also means a chatbot could produce long form videos just from text alone.

A callsheet document is also easier to use version-control software with, have you ever tried to diff ComfyUI workflow json files? Insane!

This also opens up opporunties for text modification and generating nodes and models that produce, manage or, intepret callsheet text. For example, the node pack includes a dialogue helper that will take simple dialogue lines and generate prompts for each one.

# Status

Currently this is an Alpha release.

The node interfaces may be stable, if they change older versions will be kept around until beta. And documents and workflows will be updated as changes occur.

# Installation

This node pack is in active development so you have to install it via github, either via command line or download. It also requires ffmpeg.

The current easiest way to do this is to install the [Video Helper Suite]

command line:
```
cd ComfyUI/custom_nodes
git clone https://github.com/Codes4Fun/callsheet-comfyui.git
pip install -r requirements.txt
```

or [download](https://github.com/Codes4Fun/callsheet-comfyui/archive/refs/heads/main.zip) and open the zip file and copy `callsheet-comfyui-main` to your ComfyUI installed directories `ComfyUI/custom_nodes` and rename it to `callsheet-comfyui` (without the main). And because it requires ffmpeg, the easiest solution would be to install [ComfyUI-VideoHelperSuite](https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite) which can be found in ComfyUI's extension manager.

Once this is more fully tested and stable, hopefully it can be made available directly in ComfyUI's extensions.

# Workflows

In the future workflows that integrate with more node pack dependencies may be created, for now these are the main ones:

[Krea-2 + MiniMax H3 ref2vid](workflows/callsheet_k2_mmh3.json) (depends on KJNodes)
[Krea-2 + MiniMax H3 ref2vid + Qwen Image 2.1](workflows/callsheet_k2_mmh3.json) (depends on KJNodes)

# Support

You can submit [issues on github](https://github.com/Codes4Fun/callsheet-comfyui/issues). You can `@` me in the [ComfyUI Discord dev channel](https://discord.gg/comfyorg).
