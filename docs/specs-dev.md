# Internal Specifications

The various structures used in this custom node pack.

## pipeline_spec

The pipeline specifications are currently part of the `Callsheet Text Input` node, as a json formatted string, and part of `parsed_spec` wire.

It provides information about specific pipelines. Like tasks it will handle, how to calculate resolutions and video lengths.

Example:
```
{
  "krea2": {
    "tasks": { "character":"image", "background":"image", "prop":"image" },
    "allowed_flags": [],
    "uniform_flags": [],
    "max_jobs_per_pass": 4,
    "size": {"step":8}
  },
  "minimax_h3_r2v": {
    "tasks": { "voice": "video", "video":"video" },
    "allowed_flags": [],
    "uniform_flags": [],
    "max_jobs_per_pass": 4,
    "frame_rate": 24,
    "sample_rate": 32000,
    "size": {"step":32},
    "length": {"step":17,"offset":5}
  }
}
```

The root is an object/dict with the name of the pipeline (`krea2` and `minimax_h3_r2v` in our example) and the properties of the pipeline.

`tasks` are what is handled by the pipeline from the callsheet, and the expected type: `image`, `video`, or `audio`.

`allowed_flags` all flags are allowed when empty. If specified, produces errors if a callsheet uses any flags outside of what is allowed.

`uniform_flags` the state of these specified flags must match to run in the same pass. For example if `continue_frame` is specified, that means it will avoid mixing non-`continue_frame` tasks and `continue_frame` tasks in the same pass. This is because node divergence, where a node is enabled in one task but not the other due to switch statements, can break pipeline workflows.

`max_jobs_per_pass` is really about resource management and not to blow out ram holding to much work in memory.

`frame_rate` is if the pipeline requires a specific video frame rate.

`sample_rate` is if the pipeline requires a specific audio sample rate.

`size` allows for specify default `step`, the pixel block/tile size that a pipeline requires.

`length` allows for specify default `step` and `offset` for the number of frames/samples required by the pipeline.

## item_spec

used in parsed_spec

```
{
    'index': int,
    'label': str,
    'pipeline': str,
    'type': str,
    'prompt': str,
    'negative': str,
    'width': int,
    'height': int,
    'length': int,
    'fps': int,
    'flags': [str],
    'seeds': [int],
    'injected': bool,
    'refs': [str],
    'audio_refs': [str],
    'video_refs': [[str,str]],
    'continue_frame': str,
    'continue_video': [str,str],
    'target_frame': str,
    'target_video': [str,str]
}
```

`video_refs`, `confinue_video`, `target_video` use a pair: `label` and `trim_spec` in string form. `video_refs` is a list of them.

## candidate_spec

items become candidates by resolving their refernce labels (if they have any) to keys, basically hashing an items properties and selected seed.

used in parsed pipelines but DEPRICATED and subject to removal

used internally before creating jobs

```
{
    'item' : item_spec,
    'pending': [(int, key)],
    'flags': [str],
    'ref_keys': [str],
    'aref_keys': [str],
    'vref_pairs': [[str,str]],
    'vref_windows': [(str,(int,int),int)],
    'cont_key': str,
    'cont_rec': rec_spec,
    'cv_window': (str,(int,int),int),
    'tgt_key': str,
    'tgt_rec': rec_spec,
    'tv_window': (str,(int,int),int),
    'signature': (False,)
}
```

`pending` is a list of seed and key pairs, these become jobs.

`flags` automatically has the name of the pipeline/task and reference names (including continues and targets) added, and of course user specified flags.

pairs are typically resolved key and trimspec (video trims)

windows are `(key, (start_frame,end_frame), src_fps)`


## parsed_spec

```
{
    'items': [ item_spec ],
    'selections': { label:key },
    'focus' : [label],
    'pipelines': {
        pipeline_name : pipeline_spec + {
            'candidates': [candidate_spec],
            'jobs' : [{
                'prompt': str,
                'negative': str,
                'width': int,
                'height': int,
                'length': int,
                'fps': int,
                'seed': int,
                'job': {
                    'key': str,
                    'label': str,
                    'index': int,
                    'seed': int,
                    'kind': str
                },
                'flags': [str],
                'refs': {
                    'ref_0': tensor or None,
                    'ref_1': tensor or None,
                    'ref_2': tensor or None,
                    'ref_3': tensor or None,
                    'ref_4': tensor or None,
                    'ref_5': tensor or None,
                    'ref_6': tensor or None,
                    'ref_7': tensor or None,
                    'audio_ref_0': tensor or None,
                    'audio_ref_1': tensor or None,
                    'audio_ref_2': tensor or None,
                    'audio_ref_3': tensor or None,
                    'video_ref_0': tensor or None,
                    'video_audio_ref_0': tensor or None,
                    'video_ref_1': tensor or None,
                    'video_audio_ref_1': tensor or None,
                    'video_ref_2': tensor or None,
                    'video_audio_ref_2': tensor or None,
                    'video_ref_3': tensor or None,
                    'video_audio_ref_3': tensor or None,
                    'continue_frame': tensor or None,
                    'continue_video': tensor or None,
                    'continue_video_audio': tensor or None,
                    'target_frame': tensor or None,
                    'target_video': tensor or None,
                    'target_video_audio': tensor or None
                }
            }],
            'deferred' : int
        }
    }
}
```

## rec_spec

used in the `VARIATION_STORE` and the `manifest.json` file entires

used in `candidate_spec`

image:
```
key: {
  "label": str,
  "index": int,
  "seed": int,
  "kind": "image",
  "injected": None or True,
  "filename": str,
  "subfolder": "callsheet",
  "type": "outout"
},
```
video:
```
 key: {
  "label": str,
  "index": int,
  "seed": int,
  "kind": "video",
  "injected": None or True,
  "format": str or None,
  "has_audio": bool,
  "poster": str,
  "preview": str or None,
  "filename": str,
  "subfolder": "callsheet",
  "type": "outout"
 },
```
audio:
```
  "label": str,
  "index": int,
  "seed": int,
  "kind": "audio",
  "injected": None or True,
  "format": str or None,
  "has_audio": True,
  "poster": str,
  "preview": str,
  "filename": str,
  "subfolder": "callsheet",
  "type": "outout"
```

 * `key` is the variation_key.
 * `kind` is of "image", "video", "audio".
 * `has_audio` whether a video has audio.
 * `poster` is an image of video or audio, the name in the form `f"cs_{label}_{key}_poster.jpg"`
 * `preview` is a streaming quality file, the name in the form `f"cs_{label}_{key}_preview.mp4"`
 * `filename` is the name of the saved file usually in the format `f"cs_{label}_{key}.png"`.
 * `subfolder` is usually "callsheet".
 * `type` is usually "output".
 * `format` the saved format of a video or audio file.


