# Image Pipeline From Scratch
This guide shows you how to create a simple callsheet pipeline from a standard ComfyUI template to generate character references from `1prompt`, it assumes some basic familiarity with ComyUI.

A callsheet prompt specifies the creation and editing of multiple reference assets and videos.

The basic idea of a pipeline is it handles some of those tasks in the callsheet. You can have multiple pipelines to handle multiple tasks.

A pipeline is your standard ComfyUI workflow that takes it's inputs from the callsheet and stores it's assets for reference.

## Krea-2 Template
First lets get a workflow from the ComfyUI templates. This guide uses Krea-2 but you can apply it to other text-to-image models as well.

![](images/000_TemplateKrea2Small.png)

Download and setup any missing models.

![](images/001_TemplateKrea2.png)

The nice thing about this template is everything important is already packaged up in a nice subgraph.

## Subgraph Modification

![](images/002_Krea2Subgraph.png)

We should enter the `Text to Image (Krea-2 Turbo)` subgraph.

![](images/003_Krea2Subgraph.png)

### No Previews Allowed

![](images/004_Krea2Subgraph.png)

Notice it has a `Preview as Text` node. Preview and Save nodes should be avoided since pipelines use lists that can be empty and they can produce an error as a result of a global run. So select the `Preview as Text` node and **delete** it.

Tip: If you select all nodes you plan to use in a pipeline there should not be any blue play icon in the bar above them as circled in the image.

### LoRAs

The subgraph also has built in support for LoRAs. With pipelines there are two main ways to support LoRAs with: switch nodes or with tags. The difference between these two is control of strength with switches having no way to control strength.

With that in mind, for this guide we are removing the subgraphs LoRA support, and replacing with the tag solution.

![](images/lora_003.png)

Delete the 5 LoRA related nodes in the subgraph: `Load LoRA`, `Boolean (Enable LoRA?)`, `Switch (Model)`, `If/Else Switch`, and `Concatenate Text (LoRA Trigger Word)`. Also remove the empty slots they leave behind: `enable_lora`, `lora_name`, `strength_model`, and `lora_trigger_word`.

To replace those we add a `Callsheet LoRA Tag Loader` node, which reads tags from the `prompt` in the format of `<lora:lora_name:strength_model:strength_clip>` with `:strength_clip` being optional if the LoRA includes the clip. It then removes the tag from the `prompt` in it's output. So let's see what that looks like wired up.

![](images/lora_005.png)

Connect:
 * the `model` (purple wire) from `Load Model` to the node and from the node to the `KSampler`.
 * the `clip` (yellow wire) from `Load Clip` to the node and from the node to `CLIP Text Encode (Prompt)`.
 * the `prompt` (grey wire) from `Text String (User Prompt)` to the node, and because there is a prompt enhancer, you want to wire it to the `Concatenate Text` bottom and the `If/Else Switch` `on_false` slot.

Now exit the subgraph, I recommend turning off the `prompt enhance`, setting seed to 0 and to fixed value, and run it to make sure it still works. You will also want to test that LoRA tags work, you should have a LoRA installed with the template, so you can test it by adding `<lora:krea2_darkbrush.safetensors:0.8>` to the prompt and see that it comes out different.

## Pipeline Tester

We can test how this will work in a pipeline in a full callsheet workflow by adding two nodes: `Callsheet Pipeline Tester` and `Callsheet Pipeline Basic`.

`Callsheet Pipeline Basic` is what feeds inputs into our krea-2 graph, and `Callsheet Pipeline Tester` is for testing that it works, it emulates what a callsheet parser would return.

![](images/tester_krea2.png)

Set both the nodes `pipeline` values to `krea2`, this is how our krea2 pipeline will be found. And wire their `parsed` slots together.

Move the `Resolution Selector` node over and wire it's `width` and `height` to the `Callsheet Pipeline Tester` node.

Wire `Callsheet Pipeline Basic` to the `Text to Image (Krea-2 Turbo)`: `prompt`, `width`, `height`, `seed`.

Finaly add this prompt to our tester:
```
<lora:krea2_darkbrush.safetensors:0.8>Cinematic shot of a dungeon corridor
```

And run it to test it.

Now is a good time to save this to a workflow, I suggest `callsheet_tester_krea2`.

Note: Examine the `Resolution Selector` node's advanced inputs and note down the `multiple` is `8`, for this guide I will remind you of this later.

## New Callsheet Workflow
Create a new workflow.

![](images/005_CallsheetWorkflow.png)

Add these two nodes: `Callsheet Text Input` and `Callsheet Collector` and connect `parsed` between them.

Go back to our `callsheet_tester_krea2` workflow, and select and copy the `Callsheet Pipeline Basic` and the `Text to Image (Krea-2 Turbo)` nodes into the callsheet graph, you can resize them if you want.

Connect `parsed` from `Callsheet Text Input` to `Callsheet Pipeline Basic`.

And create a `Callsheet Store Images` and connect `job` from `Callsheet Pipeline Basic`, and `images` from `Text to Image (Krea-2 Turbo)`, and `CS_ITEMS` to `items` on the `Callsheet Collector`.

IMPORTANT: For this guide, if you have not already, you should disable the subgraph's `prompt enhance` (blue when enabled), as that can alter prompts in ways which we don't always want. But this and other features can be enabled with `flags` which will be explored later.

![](images/pipeline_specs.png)

The final step is to open the advanced properties of the `Callsheet Text Input` node, this will expose a text box called `pipeline_specs`, copy this into it:
```
{
  "krea2": {
    "tasks": { "character":"image", "background":"image", "prop":"image" },
    "allowed_flags": [],
    "uniform_flags": [],
    "max_jobs_per_pass": 4,
    "size": {"step":8}
  }
}
```
Remember that `krea2` is the name we have on the pipeline, and these are settings for our pipeline. We will use this pipeline to create `character`, `background`, and `prop` images, and that is what `task` allows us to specify as well as allows us to set what the task produces which in this case is `image`. `allowed_flags` and `uniform_flags` are advanced features that we don't need for this simple workflow. `max_jobs_per_pass` is for safety that we don't store so many images in ram. And if you recall `size` and `step` represent the `multiple` value in the `Resolution Selector` in the original Krea-2 template.

And that is our first **untested** image pipeline. We can save this as `callsheet_from_scratch_krea2`.
