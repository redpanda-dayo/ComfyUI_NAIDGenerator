# ComfyUI_NAIDGenerator (Forked Version)

A [ComfyUI](https://github.com/comfyanonymous/ComfyUI) extension for generating image via NovelAI API.

## Changes in This Fork

This fork introduces the following improvements and fixes:

- Added support for `V4PromptConfig` and `V4CharacterCaption`.
- Added the `autosave_filename_prefix` property in the `GenerateNAID` node.

For more details, see the sections below.

> Example: How to generate with specified character captions

![](.github/docs/character_gen.png)

> autosave_filename_prefix

![](.github/docs/node.png)


----

## Installation

- `git clone https://github.com/bedovyy/ComfyUI_NAIDGenerator` into the `custom_nodes` directory.
- or 'Install via Git URL' from [Comfyui Manager](https://github.com/ltdrdata/ComfyUI-Manager)

## Setting up NAI account

Before using the nodes, you should set NAI_ACCESS_TOKEN on `ComfyUI/.env` file.

```
NAI_ACCESS_TOKEN=<ACCESS_TOKEN>
```

You can get persistent API token by **User Settings > Account > Get Persistent API Token** on NovelAI webpage.

Otherwise, you can get access token which is valid for 30 days using [novelai-api](https://github.com/Aedial/novelai-api).

## Usage

The nodes are located at `NovelAI` category.

![image](https://github.com/bedovyy/ComfyUI_NAIDGenerator/assets/137917911/8ab1ecc0-2ba8-4e38-8810-727e50a20923)

### Txt2img

Simply connect `GenerateNAID` node and `SaveImage` node.

![generate](https://github.com/bedovyy/ComfyUI_NAIDGenerator/assets/137917911/1328896d-7d4b-4d47-8ec2-d1c4e8e2561c)

Note that all generated images via `GeneratedNAID` node are saved as `output/NAI_autosave_12345_.png` for keeping original metadata.

### Img2img

Connect `Img2ImgOptionNAID` node to `GenerateNAID` node and put original image.

![image](https://github.com/bedovyy/ComfyUI_NAIDGenerator/assets/137917911/15ff8961-4f6b-4f23-86bf-34b86ace45c0)

Note that width and height of the source image will be resized to generation size.

### Inpainting

Connect `InpaintingOptionNAID` node to `GenerateNAID` node and put original image and mask image.

![image](https://github.com/bedovyy/ComfyUI_NAIDGenerator/assets/137917911/5ed1ad77-b90e-46be-8c37-9a5ee0935a3d)

Note that both source image and mask will be resized fit to generation size.

(You don't need `MaskImageToNAID` node to convert mask image to NAID mask image.)

### Vibe Transfer

Connect `VibeTransferOptionNAID` node to `GenerateNAID` node and put reference image.

![Comfy_workflow](https://github.com/bedovyy/ComfyUI_NAIDGenerator/assets/137917911/8c6c1c2e-f29d-42a1-b615-439155cb3164)

You can also relay Img2ImgOption on it.

![image](https://github.com/bedovyy/ComfyUI_NAIDGenerator/assets/137917911/acf0496c-8c7c-48f4-9530-18e6a23669d5)

Note that width and height of the source images will be resized to generation size. **This will change aspect ratio of source images.**

#### Multiple Vibe Transfer

Just connect multiple `VibeTransferOptionNAID` nodes to `GenerateNAID` node.

![preview_vibe_2](https://github.com/user-attachments/assets/2d56c0f7-bcd5-48ff-b436-012ea43604fe)

### ModelOption

The default model of `GenerateNAID` node is `nai-diffusion-3`(NAI Diffusion Anime V3).

If you want to change model, put `ModelOptionNAID` node to `GenerateNAID` node.

![ModelOption](https://github.com/bedovyy/ComfyUI_NAIDGenerator/assets/137917911/0b484edb-bcb5-428a-b2af-1372a9d7a34f)

### NetworkOption

You can set timeout or retry option from `NetworkOption` node.
Moreover, you can ignore error by `ignore_errors`. In that case, the result will be 1x1 size grayscale image.
Without this node, the request never retry and wait response forever, and stop the queue when error occurs

![preview_network](https://github.com/user-attachments/assets/d82b0ff2-c57c-4870-9024-8d78261a8fea)

**Note that if you set timeout too short, you may not get image but spend Anlas.**

### PromptToNAID

ComfyUI use `()` or `(word:weight)` for emphasis, but NovelAI use `{}` and `[]`. This node convert ComfyUI's prompt to NovelAI's.

Optionally, you can choose weight per brace. If you set `weight_per_brace` to 0.10, `(word:1.1)` will convert to `{word}` instead of `{{word}}`.

![image](https://github.com/bedovyy/ComfyUI_NAIDGenerator/assets/137917911/25c48350-7268-4d6f-81fe-9eb080fc6e5a)

### Director Tools

![image](https://github.com/user-attachments/assets/e205a51e-59dc-4d5a-94c8-29715ed98739)

You can find director tools like `LineArtNAID` or `EmotionNAID` on NovelAI > director_tools.

![augment_example](https://github.com/user-attachments/assets/5833e9fb-f92e-4d53-9069-58ca8503a3e7)

### V4 Support (Preview)

The node now supports NAI's V4 architecture through the nai-diffusion-4-curated-preview model. This is a preview release of V4 with some limitations:

- **Important Notes:**
  - This is a preview version of V4 and some features are limited
  - Inpainting will automatically use V3 model (but works with V4-generated images)
  - Vibe transfer is not yet supported with V4 preview (will be available with full V4 release)
  - Full V4 feature support will come with the official V4 release

#### New Model Option

NAI Diffusion V4 Curated Preview is now available in the ModelOptionNAID node:

```python
model = "nai-diffusion-4-curated-preview"
```

#### V4 Prompt Handling

Two new nodes have been added for V4 prompt handling:

##### V4BasePrompt

A node for handling V4 positive prompts:

```
V4BasePrompt -----> positive
               GenerateNAID
```

##### V4NegativePrompt

A node for handling V4 negative prompts:

```
V4NegativePrompt -> negative
                 GenerateNAID
```

#### Example V4 Workflow

Here's a basic V4 setup:

```
V4BasePrompt -----> positive
V4NegativePrompt -> negative  GenerateNAID
ModelOption ------> option
```

#### Work In Progress Features

The following V4 features are currently in development:

```python
"""
- V4PromptConfig: Advanced prompt configuration
  - Coordinate-based prompting
  - Order-based prompting
- V4CharacterCaption: Character-specific prompting with positioning
"""
```

Note: Basic img2img functionality works with V4 preview. For inpainting, the node will automatically use V3 model but can still work on V4-generated images. Vibe transfer will be supported once V4 fully releases.
