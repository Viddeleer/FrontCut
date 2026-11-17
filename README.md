# FrontCut

High quality background removal software for Windows.

![FrontCut: background replaced by a green screen](images/girl2_greenscreen_animation.png)

FrontCut cuts the main subject out of a photo with a precise, soft-edged mask, and builds on that mask to
remove or replace the background, remove the subject, blur the background, add a depth-based bokeh, or
make a colour splash. It runs entirely on your own PC (CPU, no internet connection, no uploads) and keeps
the full resolution of your photos.

It comes as a Windows program with a simple window (**FrontCut GUI**) and as a command-line program
(**FrontCut.exe**) for scripts and batch jobs.

**Features**

- **Remove background** or **remove foreground**: transparent PNG / TIF, or filled with any colour
  (e.g. a green screen).
- **Remove and inpaint object**: removes the subject and fills the hole with matching background (LaMa).
- **Background blur**: the subject stays sharp, the background gets a Gaussian blur (10 strengths).
- **Bokeh blur**: blur that grows with the depth difference to the subject, like a fast lens (10 strengths).
- **Colour splash** and **inverse colour splash**: subject in colour and background in grey, or the
  reverse; also combined with background or bokeh blur.
- **Mask** and **depth map** output, 8 or 16-bit.
- **High quality mode** (`-hr`): a 2048 x 2048 mask model for the finest hair and edge detail.
- **Edge correction** (`-e`): removes the colour fringe of the old background from hair and soft edges.
- Any image Windows can open (JPG, PNG, TIF, BMP, ...), upright by its EXIF orientation; output in
  PNG, JPG or TIF at the input's resolution.

## Requirements

- Windows 10 or 11 (64-bit) and a CPU with AVX2 (Intel 2013 / AMD 2015 or newer).
- Memory: about 12 GB free for the high quality mode (`-hr`); less without it.
- Processing takes about half a minute to a minute per image on a desktop CPU.

Download FrontCut from the [Releases](https://github.com/Viddeleer/FrontCut/releases) page and extract it
to a folder. Keep `FrontCut.exe`, `FrontCutGui.exe` and the `licenses` folder together. If Windows reports a
missing `VCRUNTIME140.dll`, install the
[Microsoft Visual C++ Redistributable (x64)](https://aka.ms/vs/17/release/vc_redist.x64.exe).

## Usage

### FrontCut GUI

1. Start `FrontCutGui.exe`.
2. Drag and drop images on the window (or on `FrontCutGui.exe`), or click **Add files...**.
3. Choose a **Function**, and the options it allows: **Strength** (blur and bokeh), **High quality (-hr)**,
   **Edge correction (-e)**, **Fill colour** (remove background / foreground) or **16-bit** (mask and
   depth map).
4. Choose the output folder (**Same as input** or a folder of your own), a **filename suffix**
   (`photo.jpg` becomes `photo_frontcut.png`) and the **format**.
5. Click **Process**. The log shows the result of every image; **Cancel** stops processing.

### Command line

```
FrontCut.exe (-m | -d | -b | -f | -i | -s | -t | [-s | -t] -g0 .. -g9 | [-s | -t] -b0 .. -b9)
             [-cRRGGBB] [-e] [-16] [-hr] <input image> <output image>
```

| Option | Function |
| --- | --- |
| `-m` | Mask (grayscale) |
| `-d` | Depth map (grayscale, near = bright) |
| `-b` | Remove the background (transparent: output `.png` or `.tif`) |
| `-f` | Remove the foreground (transparent: output `.png` or `.tif`) |
| `-i` | Remove the foreground object and inpaint the hole |
| `-s` | Colour splash: object in colour, background in grey |
| `-t` | Inverse colour splash: object in grey, background in colour |
| `-g0` .. `-g9` | Blur the background (`-g0` light .. `-g9` strong) |
| `-b0` .. `-b9` | Bokeh: blur by depth difference to the foreground object (`-b0` light .. `-b9` strong) |
| `-cRRGGBB` | With `-b` or `-f`: fill the removed part with this hex RGB colour (e.g. `-cFFFFFF` = white) |
| `-e` | Edge correction: no colour fringe at the edges (with `-b`, `-f`, `-s`, `-t`, `-g`, `-b0`..`-b9`) |
| `-16` | 16-bit mask or depth map (output `.png` or `.tif`) |
| `-hr` | High quality mask (slower; not with `-d`) |

`-s` or `-t` can be combined with `-g0`..`-g9` or `-b0`..`-b9`. Output formats: `.png`, `.tif`, `.jpg`
(not for transparent or 16-bit output). The output always has the input's resolution.

### Examples

All examples use [girl2.png](images/girl2.png). Each animation shows the original and the result.

**Green screen**: remove the background and fill it with green.

```
FrontCut.exe -b -c00FF00 -hr -e girl2.png girl2_greenscreen.png
```

![Green screen](images/girl2_greenscreen_animation.png)

**Remove object**: remove the foreground and keep the background (transparent hole). With `-i` instead of
`-f`, the hole is filled with matching background.

```
FrontCut.exe -f -hr girl2.png girl2_removeobject.png
```

![Remove object](images/girl2_removeobject_animation.png)

**Bokeh blur**: everything nearer or farther than the subject is blurred, more with more depth difference
(the animation shows two strengths).

```
FrontCut.exe -b5 -hr -e girl2.png girl2_bokeh.jpg
```

![Bokeh blur](images/girl2_bokeh_animation.png)

**Colour splash**: the subject in colour, the background in grey.

```
FrontCut.exe -s -hr -e girl2.png girl2_colorsplash.jpg
```

![Colour splash](images/girl2_colorsplash_animation.png)

**Depth map**: relative depth, near = bright.

```
FrontCut.exe -d girl2.png girl2_depth.png
```

![Depth map](images/girl2_depth_animation.png)

## How it works

- **Mask**: the withoutBG Open Weights routed pipeline. A router chooses the best model per image: the
  withoutBG matting model (run at 1344 x 1344 instead of 448 for more detail) or BiRefNet (1024 x 1024).
  With `-hr`, BiRefNet_HR runs at 2048 x 2048 for every image; its deformable convolutions run in FrontCut's
  own optimized code instead of the exported graph, which would need more than 32 GB of memory.
- **Depth**: Depth Anything V2 (part of the withoutBG model).
- **Inpainting**: LaMa, with 4x-UltraSharpV2 Lite super-resolution for large holes.
- **Edge correction**: fast foreground colour estimation (Forte & Pitie, 2021).
- **Blur and bokeh**: fast multithreaded Gaussian blur; the bokeh blurs the subject and background as
  separate layers, so no halo appears around the subject.

## Credits

Built with DINOv3. FrontCut uses these models and libraries:

| Component | Author | License | Link |
| --- | --- | --- | --- |
| withoutBG Open Weights (mask, router, depth) | Imran Kocabiyik | Apache-2.0 | [GitHub](https://github.com/withoutbg/withoutbg), [ONNX models](https://huggingface.co/withoutbg/withoutbg-openweights-onnx) |
| DINOv3 | Meta Platforms | DINOv3 License | [GitHub](https://github.com/facebookresearch/dinov3) |
| Depth Anything V2 | Lihe Yang et al. | Apache-2.0 | [GitHub](https://github.com/DepthAnything/Depth-Anything-V2) |
| BiRefNet, BiRefNet_HR | Peng Zheng | MIT | [GitHub](https://github.com/ZhengPeng7/BiRefNet), [models](https://github.com/ZhengPeng7/BiRefNet/releases/tag/v1) |
| LaMa (inpainting) | Samsung AI Center | Apache-2.0 | [GitHub](https://github.com/advimman/lama), [ONNX model](https://huggingface.co/Carve/LaMa-ONNX) |
| 4x-UltraSharpV2 Lite (super-resolution) | Kim2091 | CC BY-NC-SA 4.0 | [GitHub](https://github.com/Kim2091/Kim2091-Models/releases/tag/4x-UltraSharpV2), [ONNX model](https://huggingface.co/Kim2091/UltraSharpV2) |
| ONNX Runtime | Microsoft | MIT | [GitHub](https://github.com/microsoft/onnxruntime) |

## License

FrontCut, Copyright (c) 2026 Alain Viddeleer. Free for non-commercial use only.

The models and libraries above keep their own licenses. Their license texts and notices are in the
[licenses](licenses) folder.
