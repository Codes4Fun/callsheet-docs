# Video Prompt Basics

## Generating Voices
Now we can add voices to our characters. Add this to the end of our callsheet:
```
===

pipeline: voice
size: 0.2mp 1:1
length: 5s

===
label: ninja_voice
ref: ninja
---
subject_definitions:
<Subject 1> is ninja the young man in <Picture 1>.

summary:
[reference generation] The target video shows <Subject 1> talking.

retention_analysis:
<Subject 1> (appears in [Shot 1]): fully_preserved - everything retained.

detailed_description:
[Shot 1] <Subject 1> ninja says <d>[English] I was taught by the ninth dragon of the forest village.</d>

overall_soundscape: N/A
non_diegetic_music: N/A
===
label: ronin_voice
ref: ronin
---
subject_definitions:
<Subject 1> is ronin the middle-aged man in <Picture 1>.

summary:
[reference generation] The target video shows <Subject 1> talking.

retention_analysis:
<Subject 1> (appears in [Shot 1]): fully_preserved - everything retained.

detailed_description:
[Shot 1] <Subject 1> ronin says <d>[English] My master died in the battle of shogans, I survived.</d>

overall_soundscape: N/A
non_diegetic_music: N/A

```

Hit run ![](images/run.png), and while that generates let's review what we added.

Looking at the new callsheet text we added, we see 3 blocks. A default block that sets the `pipeline` to "voice" and `size` to 0.2 megapixels square aspect ratio. Since we are only interested in the audio we can make the video very low resolution to accelerate generation.

The two new properties being set are, `length` which can be in frames like `length: 124` or in seconds `length: 5s`. And the other property is `ref` which references the other blocks, the reference images we generated!

Tip: A general rule of thumb is 2.5 words per second average, plus padding for animations. So 5 seconds is good for about 11 words.

The prompts used are specific to MiniMax H3 reference to video model which the official guide can be found here: [VIDEO_PROMPT_WRITING_GUIDE_ref_en.md](https://huggingface.co/MiniMaxAI/MiniMax-H3/blob/main/docs/VIDEO_PROMPT_WRITING_GUIDE_ref_en.md)

But you can just copy and modify those prompts for creating other character voices.

After the run is complete we end up with 2 videos, that represent character reference voices, you can use the Collector node to find and play the videos.

![](images/211_Voices.png)

## Videos

Now that we have a taste for generating a video with a reference, we can use even more references and the new audio references for voices!

This is the complete callsheet that replaces everything in the `Callsheet Text Input` node:
```
pipeline: character
size: 1mp 1:1
===
label: ninja
---
Full body character turnaround (front, side, back) in anime style of a male ninja on plain white background. The ninja has no mask, with sharp focused eyes and determined expression. He wears form-fitting black tactical attire with sleek metallic armor plates forearms, and shins. The armor features subtle engraved patterns and clean metallic finishes. His short dark hair is slightly tousled with some strands falling across his forehead. He has several kunai knives strapped to his thighs. The illustration uses clean anime line art with consistent proportions across all angles.
===
label: ronin
---
Full body character turnaround (front, side, back) in anime style of a male ronin samurai on plain white background. The ronin has a weathered face with intense eyes and a well-groomed beard. His dark hair is partially tied in a loose topknot with some strands falling freely. He wears traditional dark blue samurai clothing without armor - a simple kimono top with wide sleeves, hakama pants, and a simple sash. A single katana is tucked into his sash, worn from frequent use. His clothing appears slightly worn but well-maintained, befitting a masterless warrior. The illustration uses clean anime line art with consistent proportions across all angles.
===
pipeline: background
size: 1mp 3:1
===
label: grocery_store
---
Cross-section view in anime background style of a grocery store. In the back is a soda isle. At the center is a table. On the right are some neatly stacked boxes.
===
pipeline: prop
size: 1mp 1:1
===
label: bottle
---
Anime style illustration of a tall transparent plastic bottle with green cap and filled with yellow liquid contents.
===

pipeline: voice
size: 0.2mp 1:1
length: 5s

===
label: ninja_voice
ref: ninja
---
subject_definitions:
<Subject 1> is ninja the young man in <Picture 1>.

summary:
[reference generation] The target video shows <Subject 1> talking.

retention_analysis:
<Subject 1> (appears in [Shot 1]): fully_preserved - everything retained.

detailed_description:
[Shot 1] <Subject 1> ninja says <d>[English] I was taught by the ninth dragon of the forest village.</d>

overall_soundscape: N/A
non_diegetic_music: N/A
===
label: ronin_voice
ref: ronin
---
subject_definitions:
<Subject 1> is ronin the middle-aged man in <Picture 1>.

summary:
[reference generation] The target video shows <Subject 1> talking.

retention_analysis:
<Subject 1> (appears in [Shot 1]): fully_preserved - everything retained.

detailed_description:
[Shot 1] <Subject 1> ronin says <d>[English] My master died in the battle of shogans, I survived.</d>

overall_soundscape: N/A
non_diegetic_music: N/A
===

pipeline: video
size: 0.5mp 16:9
length: 8s
flags: turbo

===
label: beginning
ref: ninja, ronin, bottle, grocery_store
audio_ref: ninja_voice
---
subject_definitions:
<Subject 1> is ninja the man in <Picture 1>.
<Subject 2> is ronin the man in <Picture 2>.
<Subject 3> is the bottle in <Picture 3>.
<Subject 4> is the store in <Picture 4>.
<Audio 1> is the voice-timbre reference for <Subject 1> (S1), containing a spoken English vocal layer.

summary:
[reference generation + audio reference] The target video shows an anime taking place in <Subject 4> with <Subject 2> looking at <Subject 3> and then <Subject 1> and <Subject 2> reaching for it. The shot exchange uses <Audio 1> as the voice-timbre reference for <Subject 1>.

retention_analysis:
<Subject 1> (appears in [Shot 2], [Shot 3], [Shot 5]): fully_preserved - everything retained.
<Subject 2> (appears in [Shot 1], [Shot 2], [Shot 4]): fully_preserved - everything retained.
<Subject 3> (appears in [Shot 1], [Shot 2]): fully_preserved - everything retained.
<Subject 4> (appears in [Shot 1]): fully_preserved - everything retained.
<Audio 1>: reference - its vocal timbre guides the dialogue delivery of <Subject 1> without copying the original signal.

detailed_description:
The target video is in an anime style.
[Shot 1] A low angle shot that shows <Subject 3> the bottle on the table in <Subject 4> store with <Subject 2> ronin facing away and then turning and noticing <Subject 3> the bottle, he says: "huh."
[Shot 2] Cut to a close-up shot of <Subject 3> the bottle. <Subject 1> (S1) ninja's hand and <Subject 2> ronin's hand reach for <Subject 3> the bottle and touch each other.
[Shot 3] Cut to a close-up of <Subject 1> (S1) ninja wearing a ninja mask, his eyes widen, he says: "uh!"
[Shot 4] Cut to a close-up of <Subject 2> ronin his eyes widen, he says: "uh!"
[Shot 5] Cut to a shot of <Subject 1> (S1) with ninja mask says: <d>[English] I saw it first!</d>

overall_soundscape:
sounds of a busy grocery store.

non_diegetic_music:
N/A
===
label: first_attack
ref: ninja, ronin, grocery_store
audio_ref: ninja_voice
---
subject_definitions:
<Subject 1> is ninja the man in <Picture 1>.
<Subject 2> is ronin the man in <Picture 2>.
<Subject 3> is the store in <Picture 3>.
<Audio 1> is the voice-timbre reference for <Subject 1> (S1), containing a spoken English vocal layer.

summary:
[reference generation + audio reference] The target video shows an anime taking place in <Subject 3> with <Subject 2> drawing his sword and slashing, and then <Subject 1> mask disintegrating. The shot exchange uses <Audio 1> as the voice-timbre reference for <Subject 1>.

retention_analysis:
<Subject 1> (appears in [Shot 2]): fully_preserved - everything retained.
<Subject 2> (appears in [Shot 1]): fully_preserved - everything retained.
<Subject 3> (appears in [Shot 1], [Shot 2]): fully_preserved - everything retained.
<Audio 1>: reference - its vocal timbre guides the dialogue delivery of <Subject 1> without copying the original signal.

detailed_description:
The target video is in an anime style.
[Shot 1] A shot of <Subject 2> ronin his hand reaching forward and down, then quickly moves to draw his sword and making several slices at the camera and then pointing his sword at the camera.
[Shot 2] Cut to a slow motion 3/4 view shot of <Subject 1> (S1) ninja, wearing a black face mask that disintegrates into shreds revealing a shocked expression on his face, his hands open and leaning back, a sword pointed at him.
<Subject 1> (S1) ninja then clenches his fist in anger and yells: <d>[English] YOU COWARD!</d> with the sword still pointed at him.

overall_soundscape:
sounds of a busy grocery store.

non_diegetic_music:
N/A
===
label: fair_fight
ref: ronin, ninja, grocery_store
audio_ref: ronin_voice
length: 5s
---
subject_definitions:
<Subject 1> is ronin the man in <Picture 1>.
<Subject 2> is ninja the man in <Picture 2>.
<Subject 3> is the store in <Picture 3>.
<Audio 1> is the voice-timbre reference for <Subject 1> (S1), containing a spoken English vocal layer.

summary:
[reference generation + audio reference] The target video shows an anime taking place in <Subject 3> with <Subject 1> putting his sword away. The shot exchange uses <Audio 1> as the voice-timbre reference for <Subject 1>.

retention_analysis:
<Subject 1> (appears in [Shot 2], [Shot 3], [Shot 5]): fully_preserved - everything retained.
<Subject 2> (appears in [Shot 1], [Shot 2], [Shot 4]): fully_preserved - everything retained.
<Subject 3> (appears in [Shot 1], [Shot 2]): fully_preserved - everything retained.
<Audio 1>: reference - its vocal timbre guides the dialogue delivery of <Subject 1> without copying the original signal.

detailed_description:
The target video is in an anime style.
[Shot 1] An arc shot of <Subject 1> (S1) ronin his sword pointing at <Subject 2> ninja who is seen from behind clenching his fists. <Subject 1> (S1) ronin says: <d>[English] Coward? I'm not the one who wears a mask.</d>
[Shot 2] Cut to a medium shot of <Subject 1> (S1) ronin pointing his sword forward, he says: <d>[English] But fair enough.</d>
<Subject 1> (S1) ronin sheaths his sword with metal sliding and clink sounds.
[Shot 3] Cut to a shot of <Subject 2> ninja sucker punching <Subject 1> (S1) in the face.

overall_soundscape:
silent

non_diegetic_music:
N/A
===
label: punch_out
ref: ronin, ninja, grocery_store
audio_ref: ronin_voice
---
subject_definitions:
<Subject 1> is ronin the man in <Picture 1>.
<Subject 2> is ninja the man in <Picture 2>.
<Subject 3> is the store in <Picture 3>.
<Audio 1> is the voice-timbre reference for <Subject 1> (S1), containing a spoken English vocal layer.

summary:
[reference generation + audio reference] The target video shows an anime taking place in <Subject 3> with <Subject 2> punching <Subject 1>. The shot exchange uses <Audio 1> as the voice-timbre reference for <Subject 1>.

retention_analysis:
<Subject 1> (appears in [Shot 2], [Shot 4], [Shot 6], [Shot 7], [Shot 8], [Shot 9]): fully_preserved - everything retained.
<Subject 2> (appears in [Shot 1], [Shot 3], [Shot 5]): fully_preserved - everything retained.
<Subject 3> (appears in [Shot 1], [Shot 2]): fully_preserved - everything retained.
<Audio 1>: reference - its vocal timbre guides the dialogue delivery of <Subject 1> without copying the original signal.

detailed_description:
[Shot 1] A shot of <Subject 2> (S2) ninja hits camera with right fist.
[Shot 2] A front shot of <Subject 1> (S1) ronin with his eyes closed swings his head back away from camera after being hit and says: <d>uh!</d> his head swings forward.
[Shot 3] A shot of <Subject 2> (S2) ninja hits camera with left fist.
[Shot 4] A front shot of <Subject 1> (S1) ronin with his eyes closed swings his head back away from camera after being hit and says: <d>uh!</d>.
[Shot 5] A front shot of <Subject 2> (S2) ninja drops low and comes up with an uppercut at the camera.
[Shot 6] A front shot of <Subject 1> (S1) ronin fall backwards away from camera with his arms out and says: <d>uh!</d>
[Shot 7] A side shot of <Subject 1> (S1) ronin landing on his ass.
[Shot 8] A shot of <Subject 1> (S1) ronin sitting up and wiping blood off his nose and saying with anger: <d>[English] You Sneak!</d>
[Shot 9] A close-up shot of <Subject 1> (S1) his eyes suddenly widen shocked and says: <d>huh!</d>

overall_soundscape:
N/A

non_diegetic_music:
N/A
===
label: flip
ref: ronin, ninja, grocery_store, bottle
---
subject_definitions:
<Subject 1> is ronin the man in <Picture 1>.
<Subject 2> is ninja the man in <Picture 2>.
<Subject 3> is the store in <Picture 3>.
<Subject 4> is the bottle in <Picture 4>.

summary:
[reference generation] The target video shows an anime taking place in <Subject 3> with <Subject 2> doing flips into <Subject 1> foot and is tossed into boxes of <Subject 4>.

retention_analysis:
<Subject 1> (appears in [Shot 2]): fully_preserved - everything retained.
<Subject 2> (appears in [Shot 1], [Shot 2], [Shot 3]): fully_preserved - everything retained.
<Subject 3> (appears in [Shot 1], [Shot 2], [Shot 3], [Shot 4]): fully_preserved - everything retained.
<Subject 4> (appears in [Shot 3], [Shot 4]): fully_preserved - everything retained.

detailed_description:
The target video is in an anime style.
[Shot 1] A shot of <Subject 2> ninja doing a cartwheel towards the camera that goes into a flip into the air over the camera.
[Shot 2] With <Subject 1> ronin on his back on the ground he lifts his legs up connecting his feet with the body of <Subject 2> ninja's as he lands on him.
[Shot 3] A shot of boxes in <Subject 3> with <Subject 2> ninja appearing and flying into the boxes upside down, the boxes break and <Subject 4> bottles fallout and splashing yellow liquid, and <Subject 2> ninja sitting upside on the crushed pile of boxes.
[Shot 4] A shot of <Subject 4> rolling on the floor next to a puddle of yellow liquid.

overall_soundscape:
silent

non_diegetic_music:
N/A

```
Hit run ![](images/run.png) and let's review.

You will find it has 6 additional blocks, the first of the new blocks is the default block:
```
pipeline: video
size: 0.5mp 16:9
length: 8s
flags: turbo
```
It sets `pipeline` to "video", for this demo we are using a `size` 0.5 megapixels and a 16:9 aspect ratio, and a default `length` of 8 seconds. It also now includes `flags: turbo` so that generating the 5 video clips does not take that long.

And then after the default block are the individual video blocks with labels and prompts, except one video which overrides the default length and sets it to 5 seconds, which does not affect the blocks that follow it.

You may notice a requeue, the way this system works is it only generates one or more blocks at a time, and because of references it may have to delay generations for a requeue because it depends on another asset being generated first. This creates one tricy problem where if you wish to cancel you have to clear the queue first before you can stop the active generation.

## 1video

So once the run is done you are probably noticing that all you have are clips and not a single long video, well to do that we use the `Callsheet Concat Videos` node.

![](images/213_Concat.png)

You want to set `output_name` to something relevant like `ronin_v_ninja`. The labels of the video clips we generated are `beginning,first_attack,fair_fight,punch_out,flip` enter that in the `Callsheet Concat Videos` and then you can hit run ![](images/run.png) and it will concat the clips into a complete video.

And now you have a single long video!

## Injecting and Focus

Let's say we want to replace our bottle with one from an advertisement (or an image generated from a prompt shared on discord) for [example](images/Ideogram_4.0_00372_.png).

![](images/inject_bottle.png)

To get our image into the collector we would add a standard `Load Image` node with our image and attach it to a `Callsheet Inject Image` node, setting it's label to `bottle`, and connect it's `CS_ITEMS` to `Callsheet Collector` node, and connecting it's `parsed` input to our `Callsheet Text Input` node.

We then need to update the callsheet, but let's also say we don't want to destroy the original callsheet.

![](images/replace_bottle.png)

To do that we need to setup 3 nodes and wire them together, first copy the original text to a `Text (Multiline)` node, and connect that to a `Callsheet Block Patch` node and set it's label to `bottle`, make sure `operation` is `replace`, and set it's text to:

```
label: bottle
source: injected
---
a bottle
```

It has the same label as what we are replacing but uses `source: injected` to specify that it is being injected, and the `prompt` section is more of a description of what is injected (currently required but that may change).

And finally attach `Callsheet Block Patch` node to a `Preview as Text` node and connect that to our `Callsheet Text Input` node. The purpose for the `Preview as Text` node is not only does it allow us to view and copy the text but the `Callsheet Collector` node actually reads it's contents and can refresh without needing to run by hitting it's refresh button or when switching tabs, but it does mean though hitting the play button above our `Preview as Text` when we change text.

We could hit run and it would rebuild two clips, but what if we want to only build one to make sure this work. To do that we would use **focus** on the `Callsheet Collector` node. You would first navigate our collector to find `beginning`.

![](images/focus_beginning.png)

Once you find `beginning` you would then press the target button. Notice the orange bar appears that tells you what is or are in focus (you can have multiple blocks in focus), and you can quickly navigate to them with the `goto` button, and clear them with the `clear` button.

So now if we hit run ![](images/run.png) only the `beginning` is updated, and we can make sure our changes to bottle are working, and if we are satisfied we can `clear` the focus, and run again to update the other clip that uses it, `flip`.

## Text Chain

Now let's say we want to add a cameo character to our parody, for [example](images/Shitty_Tails.png).

![](images/inject_cameo.png)

We create a new `Load Image` for our new image, and connect that to a new `Callsheet Inject Image` node we label `tails` and connect it to `Callsheet Text Input` and `Callsheet Collector` nodes.

![](images/insert_cameo.png)

And finally we add `Callsheet Block Patch` and `Callsheet Text Chain` nodes between our bottle block patch node and the `Preview as Text` node.

To the new `Callsheet Block Patch`, set it's `operation` to `insert after` and `label` to `ronin`, so that this block occurs at the end of our character definitions. And fill text with this:
```
label: tails
source: injected
---
tails
```
We label the new block being inserted as `tails` and we set `source: injected` and it's description to `tails`.

For the `Callsheet Text Chain` node we set it's text content to a new clip:
```
===
label: taken
ref: ronin, ninja, grocery_store, bottle, tails
length: 8s
---
subject_definitions:
<Subject 1> is ronin the man in <Picture 1>.
<Subject 2> is ninja the man in <Picture 2>.
<Subject 3> is the store in <Picture 3>.
<Subject 4> is the bottle in <Picture 4>.
<Subject 5> is tails in <Picture 5>.

summary:
[reference generation] The target video shows an anime taking place in <Subject 3> with <Subject 4> <Subject 2> <Subject 1> with an appearance of <Subject 5>.

retention_analysis:
<Subject 1> (appears in [Shot 1]): fully_preserved - everything retained.
<Subject 2> (appears in [Shot 1]): fully_preserved - everything retained.
<Subject 3> (appears in [Shot 1]): fully_preserved - everything retained.
<Subject 4> (appears in [Shot 1]): fully_preserved - everything retained.
<Subject 5> (appears in [Shot 1]): fully_preserved - everything retained.

detailed_description:
[Shot 1] A shot shows <Subject 4> out of focus on a table with <Subject 1> sitting up on the floor and <Subject 2> is laying on smashed boxes with transparent yellow liquid in <Subject 3>. A voice is heard in the background over a speaker system saying: <d>[English] erm, clean up on isle six</d>. <Subject 2> slides down to the ground <Subject 1> rubs his head. The focus changes to <Subject 4>, <Subject 5> appears on the left and sees <Subject 4>, smiles in excitement and takes it off the table and walks away to the left with smile and eyes closed. <Subject 1> shocked, open mouth and reaches out.

diegetic_sound:
only the speaker system is heard saying "erm, clean up on isle six", no other dialogue or voices are heard.

overall_soundscape:
indistinct crowd murmur, no foreground voices.

non_diegetic_music:
soft jazz instrumental music plays in the background of the grocery store.
```
The `Callsheet Text Chain` node basically adds the text to the end of the callsheet, making sure a new line is added in between if necessary.

The new clip is called `taken` so the final thing we want to do is add it to the end of our `Callsheet Concat Videos` node's `labels` property so that it becomes `beginning,first_attack,fair_fight,punch_out,flip,taken`, then we can run it.

