# Animation Script
A basic img2img script that will dump frames and build a video file. Suitable for creating interesting zoom in warping movies, but not too much else at this time.
The basic idea is to story board some kinf od animation with changes in prompts, translation etc at a low framerate until you get it roughly right. Then bump up the framerate for a final render, it should play out roughly the same, just more detail.

Inspired by Deforum Notebook
Must have ffmpeg installed in path.
This suffers from img2img embossing, if the image is static for too long. I would have to look at someone else's implemetation to figure out why and don't want to steal their code.

## Versions
### Original
#### Original version of the script with one basic prompt format.
### V2
#### Broken down the growing prompt into individual commands to control various parameters. Added requested features. Rolled in changed from pull requests, thanks to contributers.
### V3
#### Changed the interpolation system, so various keyframes write values into a frame dataframe. Blank values are then interpolated, so transform changes are smoother. also used to interpolated prompts by default now.
## Explanation of settings:
### Video formats:
 Create GIF, webM or MP4 file from the series of images. Regardless, .bat files will be created with the right options to make the videos at a later time.

## Total Animation Length (s):
 Total number of seconds to create. Will create fps frames for every second, as you'd expect.
### Framerate:
 Frames per second to generate.

## Denoising Strength:
 Initial denoising strength value, overrides the value above which is a bit strong for a default. Will be overridden by keyframes when they are hit.
 Note that denoising is not scaled by fps, like other parameters are.
### Denoising Decay:
 Experimental option to enable a half-life decay on the denoising strength. Its value is halved every second. Not that useful because of img2img embossing.

### Zoom Factor (scale/s):
 Initial zoom in (>1) or out (<1), at this rate per second. E.g. 2.0 will double size (and crop) every second. Will be overridden by keyframes when they are hit.
### X Pixel Shift (pixels/s):
 Shift the image right (+) or left (-) in pixels per second. Will be overridden by keyframes when they are hit.
### Y Pixel Shift (pixels/s):
 Shift the image down (+) or up (-) in pixels per second. Will be overridden by keyframes when they are hit.

## Templates:
 Provide common positive and negative prompts for each keyframe below, save typing them out over and over. They will only be applied when a keyframe is hit. The prompts in the keyframes will be appended to these and sent for processing until the next keyframe that has a prompt.

## Keyframes:
Key frames have been broken down into individual commands, since the old keyframe was blowing out.
Commands:
### prompt
Set positive and negative prompts.
Format: time_s | prompt | positive_prompts | negative_prompts
- time_s: Time in seconds from the start to make the change.
- prompt: Command name.
- positive_prompts: Replacement positive prompts. Will be concatenated with the positive template.
- negative_prompts: Replacement negative prompts. Will be concatenated with the negative template.
### transform
Set the current transform.
Format: time_s | transform | zoom | x_shift | y_shift | rotation
- time_s: Time in seconds from the start to make the change.
- transform: Command name.
- zoom: New zoom value. 1 = 100% per second. 2 = zoom in 200% over 1 second.
- x_shift: X shift value, in pixels per second.
- y_shift: Y shift value, in pixels per second.
- rotation: Rotation, in degrees per second.
### seed
Force a specific seed. It's technically a thing you can do, how usefull it is, is up to you to decide.
Format: time_s | seed | new_seed_int
- time_s: Time in seconds from the start to make the change.
- denoise: Command name.
- denoise_value: New denoise strength value.
### denoise
Set the denoise strength.
Format: time_s | denoise | denoise_value
- time_s: Time in seconds from the start to make the change.
- denoise: Command name.
- denoise_value: New denoise strength value.
### set_text
Overlay a rounded text box in post processing. I.e. only applied to the image that is saved, and it not iterated on.
Text boxes are referenced by the name you give. If you set it again, you can change the contents. Or it can be cleared. Multiple text boxes with different names can exist at the same time.
Format: time_s | set_text | textblock_name | text_prompt | x | y | fore_R | fore_G | fore_B | back_R | back_G | back_B | font_name | font_size
- time_s: Time in seconds from the start to make the change.
- set_text: Command name.
- textblock_name: Unique name or tag given to this text block in the set_text command above.
- text_prompt: Text to put in the text block. You will can use \n for multi-line.
- x: Top left X position of the text block.
- y: Top left Y position of the text block.
- fore_R: RGB values, in base10, for the text.
- fore_G: 
- fore_B: 
- back_R: RGB values, in base10, of the background bubble.
- back_G: 
- back_B: 
- font_name: name of the font file. Python will attempt to scan your system font folders for this file.
- font_size: Font size, in pixels i think.
### clear_text
Remove a named text box, it will no longer be drawn on the saved images.
Format: time_s | clear_text | textblock_name
- time_s: Time in seconds from the start to make the change.
- clear_text: Command name.
- textblock_name: Unique name or tag given to this text block in the set_text command above.
### prop
Embed a clipart image into the picture to be diffused. it will be drawn once at this time. You need to set a prop folder where transparent pngs are held, and specify them by file name.
Format: time_s | prop | prop_filename | x_pos | y_pos | scale | rotation
- time_s: Time in seconds from the start to make the change.
- prop: Command name.
- prop_filename: Name of a picture in the props folder, hopefully with transparency.
- x_pos: Center position of the prop.
- y_pos: Center position of the prop.
- scale: Scale value. 1=100% etc.
- rotation: Rotation, in degrees.
### set_stamp
Like props but applied in post processing and will not be diffused. You ca reference them by name, change their details on the fly as with text boxes.
Format: time_s | set_stamp | stamp_name | stamp_filename | x_pos | y_pos | scale | rotation
- time_s: Time in seconds from the start to make the change.
- set_stamp: Command name.
- stamp_name: Unique name or tag given to this stamp. Used to change it's parameters or delete.
- stamp_filename: Name of a picture in the props folder, hopefully with transparency.
- x_pos: Center position of the stamp.
- y_pos: Center position of the stamp.
- scale: Scale value. 1=100% etc.
- rotation: Rotation, in degrees.
### clear_stamp
Clear out a stamp, will no longer be drawn on the saved images.
Format: time_s | clear_stamp | stamp_name
- time_s: Time in seconds from the start to make the change.
- clear_stamp: Command name.
- stamp_name: Unique name or tag given to this stamp in the set_stamp command above.
### model
Allows you to change the model on the fly, if you need to. It won't change it back at the end, so if you do use this, maybe set the initial model in frame 0 first.
Format: time_s | model | model_name
- time_s: Time in seconds from the start to make the change.
- model: Command name.
- model_name: Pick one from the list. Just the name with no extension or hash is fine.
