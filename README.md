# stable_cascade_easy
Text to Img with Stable Cascade(on gradio interface), required less vram than original example on official Hugginface(https://huggingface.co/stabilityai/stable-cascade):
- 44 seconds for a 1280x1536 image with a nvidia RTX3060 with 12 GB VRAM
- 31 seconds with LCM Scheduler for a 1280x1536 image(6 steps on prior module) with a nvidia RTX3060 with 12 GB VRAM
- 26 seconds with LCM Scheduler for a 1024*768 image(6 steps on prior module) with a nvidia RTX3060 with 12 GB VRAM
  
![](src/screenshot.png)

## Why is stable_cascade_easy faster than hugginface example of stability ai?
Answer: because stable cascade is composed of two models, many gb each, stability ai example loads both models simultaneously into the gpu vram. While this application loads the first one(prior), creates the image(latents), cleans the vram and sends the image(latents) to the second model(decoder) and then returns the final image and cleans the vram completely... for PC with less than 16 gb of vram without this "trick" all 2 models would not fit in the vram and then you would have to use the system ram with a huge drop in performance(the time drops from 10 minutes to 44 seconds, 1280x1536 with nvidia rtx 3060 12 gb vram)

# Versions:
- v1.0: First version
- v1.1: Diffusers fix
- v1.2: Scheduler drop down menu(with LCM and DPM++ 2M Karras compatibility)
- v1.2.5: Added "Batch Size", number of images per prompt at the same time

# Diffusers
The diffusers branch is currently broken, meanwhile you can install it from an older commit(--force needed):
 ```bash
.\venv\Scripts\activate
pip install git+https://github.com/kashif/diffusers.git@a3dc21385b7386beb3dab3a9845962ede6765887 --force
 ```

# Installation:
1. Install [Python 3.10.6](https://www.python.org/downloads/release/python-3106/), checking "Add Python to PATH".
2. Install [git](https://git-scm.com/download/win).
3. On terminal:
```bash
git clone https://github.com/shiroppo/stable_cascade_easy
cd stable_cascade_easy
py -m venv venv
.\venv\Scripts\activate
pip install -r requirements.txt
pip install git+https://github.com/kashif/diffusers.git@a3dc21385b7386beb3dab3a9845962ede6765887 --force
```
# Run:
### Method 1
Double click on ```app.bat``` on stable_cascade_easy directory
### Method 2
On terminal:
```bash
.\venv\Scripts\activate
py app.py
```
# Update:
1. ```git pull```(if error: ```git stash``` and after ```git pull```)
2. ```.\venv\Scripts\activate```
3. ```pip install -r requirements.txt```

# Scheduler
You can choose between DDPMWuerstchenScheduler(default), DPM++ 2M Karras and LCM. Euler a and DPM++ SDE Karras create errors so it can't be selected, scheduler only for prior model, decode model only works with default scheduler.

## Scheduler - DDPMWuerstchenScheduler(default)
Default scheduler, guidance scale recommended: 4, prior steps recommended: 20+

## Scheduler - DPM++ 2M Karras
Sometimes better results than DDPMWuerstchenScheduler(default), guidance scale recommended: 6+, prior steps recommended: 20+

## Scheduler - LCM
LCM can use 6+ steps on prior models so the image creation is even faster, guidance scale recommended: 4, prior steps recommended: from 6 to 18

# Output
Created images will be saved in the "image" folder

## Contrast:
Possibility to change the final image contrast, value from 0.5 to 1.5, no change with value 1(best results from 0.95 to 1.05)

## Dimensions(Width and Length)
Multiples of 128 for Stable Cascade, but the app will resize the image for you, so you can choose any size you want

## Guidance Scale and Guidance Scale Decode
Choice the value that you want for Guidance Scale(Prior), for the Guidance Scale Decode now is hidden because value different than 0 causes errors and consequent not creation of the image
  
## Code(without gradio):
```bash
import torch
from diffusers import StableCascadeDecoderPipeline, StableCascadePriorPipeline
import gc

device = "cuda"
num_images_per_prompt = 1

prior = StableCascadePriorPipeline.from_pretrained("stabilityai/stable-cascade-prior", torch_dtype=torch.bfloat16).to(device)
prior.safety_checker = None
prior.requires_safety_checker = False

prompt = "a cat"
negative_prompt = ""

prior_output = prior(
    prompt=prompt,
    width=1280,
    height=1536,
    negative_prompt=negative_prompt,
    guidance_scale=4.0,
    num_images_per_prompt=num_images_per_prompt,
    num_inference_steps=20
)

del prior
gc.collect()
torch.cuda.empty_cache()

decoder = StableCascadeDecoderPipeline.from_pretrained("stabilityai/stable-cascade",  torch_dtype=torch.float16).to(device)
decoder.safety_checker = None
decoder.requires_safety_checker = False

decoder_output = decoder(
    image_embeddings=prior_output.image_embeddings.half(),
    prompt=prompt,
    negative_prompt=negative_prompt,
    guidance_scale=0.0,
    output_type="pil",
    num_inference_steps=12
).images[0].save("image.png")

# del decoder
# gc.collect()
# torch.cuda.empty_cache()
```
## Support:
- ko-fi: (https://ko-fi.com/shiroppo)
