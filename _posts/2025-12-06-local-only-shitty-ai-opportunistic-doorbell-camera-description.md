---
layout: post
title: My Local-only shitty AI opportunistic doorbell camera explainer explainer
date: 2025-12-06 10:00:00 -0500
type: post
published: true
status: publish
description: Self-hosted low-end AI (high-end gaming GPU) to give some laughs
keywords: 
categories:
- self-hosted
- AI
- shitty-AI
- home-assistant
- fun
tags:
- AI
- shitty-AI
- home-assistant
- fun
---
{%- raw %}
tl; dr: For today at least, I use Home Assistant's HACS integrations for vivint (my home security system), ollama-vision (I don't know wtf this means), and my *Windows* gaming PC which has an RTX 5080 (16GB VRAM) to give a slow, inaccurate, and unreliable description of who or what is at my front door based on AI magicks'ing my doorbell camera. It takes around 3 seconds usually, sometimes far less (600ms) and sometimes far more (6 fucking minutes). But it is 100% completely self-hosted and bereft of subscription fees, though thus wildly unreliable.

Amend this classic image:
![somebody at my front door](/assets/home-assistant-doorbell-mailman.jpg){: width="250" }

With a follow-up notification describing them:
![OMG, it's the mail dude! He's got a big ol' mail bag on his back and he's tryin' to get the mail in the box. He's wearin' a blue postal uniform with a white hat and REDACTD that's like totally on point. He's got REDACTED](/assets/home-assistant-doorbell-text.jpg){: width="250" }


While **100% self-hosted on your own network** on mostly open source stuff. On Linux the closed-source NVIDIA drivers are required for ollama to be useful.

I don't know what MCP is, I don't know what "local inference" means and I don't give a shit and that's my problem professionally, but not personally. I want shitty automated jokes about people at my door delivered to my phone within a few mintues maybe. I don't know in any detail what `llama3.2-vision` means and how it compares to anything else listed on ollama models lists. I'm not a luddite, I'm just a human bean and not super excited about techno-fascists sending obviated labor to prisions via a few trivial intermediate steps.

1. Run Home Assistant on a NUC in your closet or garage, your "MPoE." Don't think about it.
2. Have fun gaming on a PC with a "decent" (read: great) GPU.
3. Use your PC space heater to give some laughs.

## Get to the point:
I'd been considering buying an additional mini PC from my ~$1K ASUS NUC 14 Pro with 128G-fuckin-B of RAM, a ~$2K AMD Ryzen AI Max+ 395 system like the Framework Desktop due to its unified memory, giving the integrated GPU something like 72G-fuckin-B of **VRAM**. But I already have a gaming PC with which also cost me $2K, so why can't I try that paltry 16GB VRAM GPU of an RTX 5080? It works. Poorly and inaccurately, but it totally works well enough to give me a laugh instead of another headache.

The pieces, I think:
1. Run Home Assistant on some rando place, nobody cares where just do it it's fun to make your lights turn off at bedtime.
2. Enjoy gaming on a PC and forego the budget anguish of 2025 PC gaming with 60%+ of the build cost being solely the GPU.
3. Have a doorbell camera or some other fuckin' camera, I don't give a shit. This is obviously not doorbell specific.
4. Resign to the fact that you are not gaming on your PC as much as you thought you were.
5. Use that now-slack capacity of your GPU just in case your cameras need some shitty AI text to accompany them.
6. Be a bit more patient, you're using your shit-ass home server and your shit-ass gaming PC's maybe-not-otherwise-utilized GPU.

##  Run Home Assistant on some rando place
Nobody cares where just do it it's fun to make your lights turn off at bedtime. I'm not Google, figure this out on your own.

## Enjoy gaming on a PC
This is increasingly difficult with hardware spread tripping myriad bugs, component costs blowing through every ceiling, and your day job also being glued to the same chair, keyboard, monitor, and mouse. It's a fuckin' drag.

## Have a camera to en-AI the things
I don't give a shit, if you're here you already have one because you're me and nobody reads this.

## Admit that your gaming PC's GPU is largely unutilized
Get over it, you know I'm right. If I'm not then congrats on your retirement.

## Self-Hosted GPU for Home Assistant ollama-vision summary of your doorbell camera
i.e. the point of this post, why did I write other shit? What an idiot.

***First:*** [Download ollama](https://ollama.com/) onto your gaming PC. Play around with it. It's fun. Try to make it swear, try to make it say "boner" and all that. Have your fun.

Try out a "vision model." I don't know what that means other than I can supply an image URL from wikipedia or wherever and ask it to describe the image and it does. I can ask it to make fun of the image and it mostly does. I tried "llama3.2-vision" because 1) it says "vision", and 2) "ollama" shares a lot of in-sequence letters with "llama" so I suppose they're kind of thematically related (yes, I get that my employer, Meta, made LLAMA but that's honestly all I understand as the similarity).

***Second:*** Try out the [Home Assistant integration for ollama](https://www.home-assistant.io/integrations/ollama/) pointing back to your gaming PC. Run it with this command in a terminal.
PowerShell:
```
$Env:OLLAMA_HOST = "0.0.0.0:11434"; ollama serve
```
bash:
```
OLLAMA_HOST=0.0.0.0:11434 ollama serve
```

Then configure the integration to point to your gaming PC's IP. Talk to it, use it as a chatbot. It works, it's not amazing, but it's yours. Use it as a "conversation agent" for HA's "Assist" instead of their own or OpenAI. Ollama is "free" as in "it's winter and my energy bill goes to heat anyway, so sure power up that RTX 5080 space heater."

***Third:*** Throw away the ollama integration. It was fun, but that's not why you're here. You're here to automate, not to chat. Use the [ollama_vision](https://github.com/remimikalsen/ollama_vision/) integration in HACS. And *VERY CAREFULLY* read the section on "Events": [https://github.com/remimikalsen/ollama_vision/?tab=readme-ov-file#events](https://github.com/remimikalsen/ollama_vision/?tab=readme-ov-file#events). Point it at your gaming PC. If you don't know what model to use, neither does anybody else except for the mega fans. NOBODY ELSE CARES, folks. I picked llama3.2-vision and it works. I don't know why, but it works. I suppose it's because it says "llama" and "vision?" I don't care.

***Fourth:*** Create an automation to *manually* send a [Wikipedia hotdog image](https://en.wikipedia.org/wiki/Hot_dog#/media/File:Hot_dog_with_mustard.png) to ollama-vision when you *manually* force the automation to run. Play with it. Change the URL around a lot. Try the not-a-hotdog thing, whatever. Get this working first. In order to do so, you'll need 4 things open at once: ollama in powershell, Home Assistant automation editor, **Home Assistant event subscribing to `ollama_vision_image_analyzed`** (not obvious in any docs), and the Home Assistant logs. For the Home Assistant automation you'll want something like this for the full automation:
```
alias: hotdog
description: ""
triggers: []
conditions: []
actions:
  - action: ollama_vision.analyze_image
    metadata: {}
    data:
      prompt: Is this a hotdog?
      image_url: >-
        https://www.w3schools.com/html/pic_trulli.jpg
      device_id: XXXXXXXXXXXXXXXXXXXXXX
      image_name: hotdog

```
Hint: that wikipedia hotdog URL doesn't work but https://www.w3schools.com/html/pic_trulli.jpg does and it's *not* a hot dog nor a penis.

Errors I hit:
* Home Assistant logs showed an HTTP 403 error when fetching an image by URL (like the Wikipedia hotdog image). Solution: use a different test image URL.
* Formatting yaml is fuckin' dumb so of course everything is always wrong at first. Solution: git gud.
* Can't see how it all hops across from manual press -> image fetch -> ollama GPU stuff -> notification. Solution: really seriously open all 4 of those things in one screen, 1/4 each.

***Fifth:*** Once you have not-a-hotdog working, you should see something in the Home Assistant event subscription UI on `ollama_vision_image_analyzed` like this:
```
event_type: ollama_vision_image_analyzed
data:
  integration_id: XXXXXXXXXXXXXXXXXXXXXX
  image_name: hotdog
  image_url: https://www.w3schools.com/html/pic_trulli.jpg
  prompt: Is this a hotdog?
  description: >-
    No, this is not a hotdog. This is a picture of a traditional Italian village
    called Alberi, located in the region of Pugli in Italy. The village is known
    for its unique architecture and is often referred to as the "trulli"
    village. The trulli are small, round, and cone-shaped houses made of stone
    and are typically found in this region. The village is also known for its
    beautiful scenery and is a popular tourist destination.
  used_text_model: false
  text_prompt: null
  final_description: >-
    No, this is not a hotdog. This is a picture of a traditional Italian village
    called Alberi, located in the region of Pugli in Italy. The village is known
    for its unique architecture and is often referred to as the "trulli"
    village. The trulli are small, round, and cone-shaped houses made of stone
    and are typically found in this region. The village is also known for its
    beautiful scenery and is a popular tourist destination.
origin: LOCAL
time_fired: "2025-12-07T05:55:21.819316+00:00"
context:
  id: XXXXXXXXXXXXXXXXXXXXXXXXXXXXX
  parent_id: null
  user_id: null
```
Here your important information for the notification is "image_name" and "final_description". You set "image_name" in the ollama-vision integration.

***Sixth:*** Make an automation which triggers on your doorbell or motion, don't care, and takes a camera snapshot. Then send that camera snapshot over to ollama-vision.
For example, the full YAML for this trigger->ollama-vision automation is:
```
alias: Poke ollama vision
description: ""
triggers:
  - trigger: state
    entity_id:
      - binary_sensor.hotdog_motion
    from:
      - "off"
      - "on"
conditions: []
actions:
  - action: camera.snapshot
    metadata: {}
    data:
      filename: /config/www/snapshots/hotdog.jpg
    target:
      entity_id: camera.doorbell
  - action: notify.mobile_app_your_phone_name
    metadata: {}
    data:
      message: Motion at the hotdog
      data:
        image: /local/snapshots/hotdog.jpg
  - action: ollama_vision.analyze_image
    metadata: {}
    data:
      prompt: Is that a hotdog?
      image_url: >-
        http://homeassistant.lan:8123/local/snapshots/hotdog.jpg
      device_id: xxxxxxxxxxxxxxxxxxxxx
      image_name: hotdog
mode: single
```
This does a few things:
1. triggers when the hotdog camera motion binary sensor switches from off to on (when there is motion detected).
2. without condition (not a great idea)
3. snapshot the camera to *a location which is AN OPEN URL FOR ANYTHING AUTHENTICATED TO HOME ASSISTANT (also not a great idea)*
4. Notify your phone with that image so you can refer to it later
5. Huck it off to your gaming PC's ollama to hopefully maybe describe it unless you're gaming.

***Seventh:*** This is actually *separately*, make another automation which listens to `ollama_vision_image_analyzed` and unpacks the `trigger.event.*` info. Again, an example of the event data itself is:
```
event_type: ollama_vision_image_analyzed
data:
  integration_id: XXXXXXXXXXXXXXXXXXXXXXXX
  image_name: hotdog
  image_url: https://www.w3schools.com/html/pic_trulli.jpg
  prompt: Is this a hotdog?
  description: >-
    No, this is not a hotdog. This is a picture of a traditional Italian village
    called Alberi, located in the region of Pugli in Italy. The village is known
    for its unique architecture and is often referred to as the "trulli"
    village. The trulli are small, round, and cone-shaped houses made of stone
    and are typically found in this region. The village is also known for its
    beautiful scenery and is a popular tourist destination.
  used_text_model: false
  text_prompt: null
  final_description: >-
    No, this is not a hotdog. This is a picture of a traditional Italian village
    called Alberi, located in the region of Pugli in Italy. The village is known
    for its unique architecture and is often referred to as the "trulli"
    village. The trulli are small, round, and cone-shaped houses made of stone
    and are typically found in this region. The village is also known for its
    beautiful scenery and is a popular tourist destination.
origin: LOCAL
time_fired: "2025-12-07T05:55:21.819316+00:00"
context:
  id: XXXXXXXXXXXXXXXXXX
  parent_id: null
  user_id: null
```
This is a separate animation because it can take an unpredictable amount of time ranging from 600ms to 60000ms (1 minute).

In this additional automation you grab the event data and put it into a notification. Use `trigger.event.data.*` fields in conditions and message content/title:

{% raw %}
```
alias: notify about doorbell AI roast
description: ""
triggers:
  - event_type: ollama_vision_image_analyzed
    trigger: event
conditions:
  - condition: template
    value_template: |
      {{ trigger.event.data.image_name == "hotdog" }}
actions:
  - data:
      message: |
        {{ trigger.event.data.final_description }}
      title: Hotdog found on camera? News at 11.
    action: notify.mobile_app_your_phone
mode: single
```

## Be patient, relax your expectations
As cool as it seems to get a phone notification with a custom-prompted vision-llm AI bot telling you caustic or pithy quips to your phone, please rember that your wallet has been doing just fine without that for a good 100,000 fucking years.

Remember that modern AI's hallucinate all the time and so it's dangerous to rely on their output for anything important like physical safety.

Remember that you don't actually give a shit if it takes 2s versus 2 minutes to get a shitty AI generated joke about what's at your door or on your camera. The point is the laugh, not the latency. Don't pay for latency when what you want is the laugh.

Do you care about what an AI describes at your camera while you're playing your game, using your GPU? No. If you do: no you don't, shut up.

{% endraw %}
