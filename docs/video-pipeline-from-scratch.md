# Video Pipeline From Scratch
This guide is a continuation of the previous guide, [Image Pipeline From Scratch](#image-pipeline-from-scratch). If you did not save the workflow from that guide you can get a starting workflow here: [callsheet_from_scratch_k2.json](workflows/callsheet_from_scratch_k2.json)

If all we wanted to do was generate reference images we would be done, but in this guide series our goal is really to generate long form videos!

For this next part it is highly recommended you install Kijai's custom node pack found in the ComfyUi Node Manager as `ComfyUI-KJNodes`. The node we want is called `Model Preview Override` and you will also need [taeh3.safetensors](https://huggingface.co/Kijai/MiniMax-H3-TAE/resolve/main/vae_approx/taeh3.safetensors?download=true) to be in your `ComfyUI/models/vae_approx` folder. The reason for this is that it makes you more productive by allowing you to watch the generation and stop it early if it looks wrong and make changes to the prompt.

## Template
Just like with Krea-2 in the previous pipeline, we will start with the MiniMax H3 Reference to Image template:

![](images/200_TemplateMiniMaxH3.png)

Download and setup any missing models.

## Workflow Modification

![](images/201_TemplateMiniMaxH3.png)

The first thing we notice is that, unlike the Krea-2 template, there is no subgraph, so we will want to create one! But we should note it seperates the input nodes at the bottom from the rest, and for our subgraphs we won't be needing those or the `Create Video` and `Save Video` nodes.

The next step then we add two nodes: `Model Preview Override` and `Callsheet LoRA Tag Loader` nodes. Let's see what that looks like wired up:

![](images/202_PreviewOverride.png)

Connect:
 * the `model` purple slots from `If/Else Switch (model)` to `Model Preview Override` to `Callsheet LoRA Tag Loader` to `Basic Guider` and `BasicScheduler`.
 * the `clip` yellow slots from `Load CLIP` to `Callsheet LoRA Tag Loader` to `MiniMax H3 Reference to Video`.
 * the `prompt` grey slots from `Input Text (Prompt)` to `Callsheet LoRA Tag Loader` to `MiniMax H3 Reference to Video`.

For the `Model Preview Override` be sure to select the `taeh3.safetensors` for the `tiny_vae` property, set `preview_frames` to `240`, and you might as well resize the node to fill in the gap and increase the preview area.

Now we should run it and make sure it works!

With it working we want to select 6 nodes and create a subgraph:

![](images/204_ModelSubgraph.png)

That will be our model subgraph with a turbo switch, so let's rename the subraph to `MMH3 Model` and the switch slot to `turbo` and outputs to `model` and `steps`.

![](images/205_ModelSubgraph.png)

Now we want to create a condition/sample subgraph by selecting the 12 other nodes, not the nodes in the `User Input` blue frame and not the `Create Video` node, not our new `MMH3 Model` subgraph, nor the `Model Preview Override` node.

![](images/206_SampleSubgraph.png)

So select `Load Clip`, both `Load VAE`, `MiniMax H3 Reference to Video`, `Callsheet LoRA Tag Loader`, `RandomNoise`, `Basic Guider`, `KSamplerSelect`, `BasicScheduler`, `SamplerCustomAdvanced`, `VAE Decode`, and `VAE Decode Audio`, and create a subgraph and name it `MMH3 Sampler`.

![](images/207_SampleSubgraph.png)

This gives us 3 important nodes, the `MMH3 Model` subgraph, the `Model Preview Override` node and the `MMH3 Sampler` subgraph.

## Pipeline

The MiniMax H3 Reference to Video model supports references and to take advantage of those references there are pipeline nodes for references. So to start we should enter the `MMH3 Sampler` subgraph.

![](images/video_pipeline_tester_001.png)

We are about to add a lot of nodes, so I suggest rearranging the nodes as pictured, with loaders in the top left, followed by `MiniMax H3 Reference to Video`, and space between the 5 sampler nodes and the decoders. You also might want to fix the output slots if they were made out of order like mine were (be sure to fix the `Create Video` outside of the subgraph if you do). For subgraph inputs, delete the two `ref_image_` slots as they will be replaced, and attach the `RandomNoise` node. This can be somewhat buggy, make sure the other inputs like `width` `height` `length` are attached correctly and fix them if they are not.

For our next step we will create four nodes: `Callsheet Image Refs`, `Callsheet Video Image Refs`, `Callsheet Video Audio Refs`, and `Callsheet Audio Refs`, placed in front of our `MiniMax H3 Reference to Video` node.

![](images/video_pipeline_tester_002.png)

Start by connecting `Callsheet Image Refs` `refs` slot to the subgraph input slot, and then connect the rest of the nodes `refs` to that subgraph input slot.

Then connect all of `Callsheet Image Refs` `ref_` slots to `ref_image_` slots of `MiniMax H3 Reference to Video`, which will expand adding slots as you do this. And connect only 3 from `Callsheet Video Image Refs` `video_ref_` to `ref_video_`, `Callsheet Video Audio Refs` `video_audio_ref_` to `ref_video_audio_`, and `Callsheet Audio Refs` `audio_ref_` to `ref_audio_`.

Since we are there, for `MiniMax H3 Reference to Video`, change `ref_image_size` from `match` to `max` to get higher quality references.

### First and Last frames

Pipelines supports first and last frames so we want to add first and last frame support to our subgraph. The problem is we need to use separate nodes to do so and those nodes produce errors if no images exists. So to achieve that we need nodes to detect there is no image and controls a switch. For the last frame we also need to know how many frames there are and set the frame index to it's negative (negative numbers represent an index from the end of the video instead of it's start).

![](images/ref_first_last_guide_001.png)

So we will start by adding `If/Else Switch`, `Add Guide for MiniMax H3`, and `Callsheet Has Value`, and connect `Add Guide for MiniMax H3` nodes `positive` out to the `If/Else Switch` nodes `on_true`, and then the `Callsheet Has Value` nodes `has_value` to the switch on the `If/Else Switch` node. Then copy these nodes to the right of them, connecting the `output` of the switch on the left to `Add Guide for MiniMax H3` node's `positive` slot and to the switch on the rights `on_false` slot.

Then add a `Get Image Size` node and a `Math Expression` node, connecting `batch_size` to `a`. Set the math `expression` to `-a`, and connect it's `INT` to the `frame_idx` of `Add Guide for MiniMax H3` node on the right.

Select these new nodes and create a subgraph. Connect all slots of the left `Add Guide for MiniMax H3` to the subgraph input, rename the slots `image` to `first_frames` and `audio` to `first_audio`. Connect the left switch's `on_false` slot to the subgraph `positive` slot. Connect the left `Callsheet Has Value` node's `value` slot to `first_frames`. Connect up the right `Add Guide for MiniMax H3` nodes `latent`, `vae`, and `audio_vae` to the existing subgraph slots, but create new subgraph slots for it's `image` and `audio` but rename those new slots `last_frames` and `last_audio`. Connect the new `last_frames` slot to the right `Callsheet Has Value` nodes `value` and to `Get Image Size` nodes `image` slot.

![](images/ref_first_last_guide_002.png)

Exit the subgraph and a name it `Add First/Last Guide MMH3`. Connect `MiniMax H3 Reference to Video` node's `positive` and `latent` to the new add guide subgraph, and connect the new add guide subgraph's `output` to the `Basic Guider` node's `conditioning`. Connect the VAEs from the loaders to the subgraph.

Finally create a `Callsheet First/Last Refs` node, connect it's `refs` slot to the subgraph's `refs` slot like the other ref nodes, and connect it's outputs to their corresponding inputs on our new add guide subgraph.

Exit the `MMH3 Sampler` subgraph.

### Pipeline Tester

Add 3 nodes: `Callsheet Pipeline Tester`, `Callsheet Pipeline Basic`, and `Callsheet Flag`.

![](images/video_pipeline_tester_003.png)

Connect `Callsheet Pipeline Tester` node's `parsed` to `Callsheet Pipeline Basic`. Set the `pipeline` name to `mmh3_r2v` for both nodes.

Connect `Callsheet Pipeline Basic` node's `prompt`, `width`, `height`, `length`, `seed`, and `refs` to the `MMH3 Sampler` subgraph.

For the turbo LoRA switch we use pipeline flag, so connect `Callsheet Pipeline Basic` node's `flags` output to `Callsheet Flag` and it's `present` to the `MMH3 Model` subgraph's `turbo` switch. Set `Callsheet Flag` node's `flag` to `turbo`.

For testing connect up the templates old input node's to our `Callsheet Pipeline Tester`, the `prompt`, `width`, `height`, `length`, and the two `ref` images, also set it's `flags` value to `turbo`.

Run your test and make sure there are not errors. You can also try disconnecting the reference images and connecting them to `continue_frame` and `target_frame` slots to make those work. You can try with and without the `turbo` flag in the tester as well.

Now is a good time to save this workflow to `callsheet_tester_mmh3`.

We should note that the `Resolution Selector` node uses a `multiple` of `32`, higher than what we saw with Krea-2, I'll remind you of this later.

## Callsheet Workflow

From our `callsheet_tester_mmh3` workflow copy 5 nodes: `MMH3 Model` subgraph, `Model Preview Override` node, `Callsheet Pipeline Basic` node, `Callsheet Flag` node, and `MMH3 Sampler` subgraph.

Open our `callsheet_from_scratch_k2` workflow, if you don't have it you can [download it](workflows/callsheet_from_scratch_k2.json). Paste the new nodes in it near the bottom.

![](images/video_pipeline.png)

Connect `parsed` between `Callsheet Text Input` and `Callsheet Pipeline Basic` of our new nodes.

Create a `Callsheet Store` node, connecting `job` and `fps` from `Callsheet Pipeline Basic` node, and connecting `image` and `audio` from `MMH3 Sampler` subgraph. And connect it's `CS_ITEMS` output to the `Callsheet Collector` node's `items_2` slot.

Create a `Callsheet Concat Videos` node below our collector node, connecting `parsed` to the `Callsheet Text Input` and `after_1` to the `CS_ITEMS` of the new `Callsheet Store`.

Next we need to update our `pipeline_specs` on the `Callsheet Text Input` node's advanced properties with:

```
{
  "krea2": {
    "tasks": { "character":"image", "background":"image", "prop":"image" },
    "allowed_flags": [],
    "uniform_flags": [],
    "max_jobs_per_pass": 4,
    "size": {"step":8}
  },
  "mmh3_r2v": {
    "tasks": { "voice":"video", "video":"video", "image_edit":"image" },
    "allowed_flags": [],
    "uniform_flags": ["continue_frame","continue_video","target_frame","target_video"],
    "max_jobs_per_pass": 4,
    "frame_rate": 24,
    "sample_rate": 32000,
    "size": {"step":32},
    "length": {"step":17,"offset":5}
  }
}
```

For `tasks` this pipeline it handles threee. Two "video" tasks and one "image" task. We could technically set the "voice" task to produce an "audio" track only but it is kind of nice seeing the character say the lines, so here we have made it a "video".

The `uniform_flags` are to prevent tasks that use any of those properties to not be done at the same time as ones with different properties. This is because of the switches we had to add in the middle of the graph for the first/last frames. The switches have to be in the same state for tasks to run simultaniously. This does not apply to our turbo switch because it affects something that exists as an input to the graph and not in the graph itself.

We set `frame_rate` and `sample_rate` to what MiniMax H3 works with. `frame_rate` will be used to convert from seconds to frames in callsheets.

`size` `step` is what I said we should remember from the `Resolution Selector` node's `multiple` which was `32`.

The `length` is the stepping and offset of what is used internally in MiniMax H3.

And this is our yet to be fully tested callsheet video pipeline. Now is a good time to save this workflow as `callsheet_from_scratch`.
