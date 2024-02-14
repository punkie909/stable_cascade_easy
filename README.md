# stable_cascade_easy
Text to Img with Stable Cascade, required less vram than original example on official Hugginface:
- 60 seconds for a 1280x1536 image with a nVidia RTX3060 with 12 GB VRAM
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
```
- and then:
```bash
pip install git+https://github.com/kashif/diffusers.git@wuerstchen-v3
```

# Code:
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
    num_inference_steps=30
).images[0].save("Image.png")

# del decoder
# gc.collect()
# torch.cuda.empty_cache()
```
