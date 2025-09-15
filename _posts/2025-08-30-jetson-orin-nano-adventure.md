---
layout: post
title: "Jetson Orin Nano Adventures"
date: 2025-08-30 09:00:00 -0400
categories: [jetson]
tags: [Jetson, Orin Nano, CUDA, L4T, Ubuntu]
description: "Jetson Orin Nano setup from scratch: flash JetPack, enable CUDA/cuDNN, verify GPU, fix the usual errors."
image: /assets/img/posts/jetson-orin-nano-setup/cover.png
images_path: /assets/img/posts/jetson-orin-nano-setup
---

## Weekend Hack Project

Balancing work, coaching/playing rugby, professional development, and my personal life is always a bit of a struggle. Each plays a pivotal role in my mental/physical health and the consistency of my day-to-day life. Finding time for all of it is as important as having a healthy balanced diet.

Like any sport, rugby is generally seasonal, so I often find time during the offseason to focus on professional development and hobbyist ventures. This year, in addition to studying for the AWS DEA exam, I decided to step back into the world of edge computing. After some _light_ research, I decided to order an edge compute device and patiently waited months until it landed on my doorstep.

This weekend I FINALLY booted up my **Jetson Orin Nano Developer Kit** — and let’s just say it was less “plug-and-play” and more “learn-by-breaking.” 😅 To start, I flashed a 256GB SD Card with balena etcher, inserted the image, plugged in my peripherals, and kicked off the install. After a few reboots, all systems were go ✅. **\[**[Get-started-jetson-orin-nano-devkit](https://developer.nvidia.com/embedded/learn/get-started-jetson-orin-nano-devkit#setup)**\]**

![JTOP Snap]({{ page.images_path }}/jtop-jetson.png)

### **Pt 1: Start simple: _“run the demo repo.” ⏯️_**

Once I was able to access the GUI, I decided to pursue _image detection_ in the **\[**[**jetson-inference**](http://github.com/dusty-nv/jetson-inference)**\]** repo for a quick start. The jetson-inference repo is designed to guide users in deploying, using, and training inference and real time vision models. It seemed like the best place to get my toes wet without wading too deep in the tech. I excitedly followed the guide to set up the imagenet image classification model. Everything was installed, and I was set to run. I quickly mashed together the command in the terminal, ENTER, and of course … it failed.

`$ cd jetson-inference/build/aarch64/bin`  
`$ ./imagenet.py images/orange_0.jpg images/test/output_0.jpg`

**Key-Takeaway 1**: Jetson AI workflows aren’t pure Python — they sit on top of carefully compiled GPU libraries. You don’t get that until you go through the motions. Not as ‘plug and play’ as I imagined.

### **Pt 2: The First Hurdle** 🚧

**ERROR: TensorRT 10.3 doesn’t support legacy Caffe models**.

After receiving the first failure, I spent a few hours chasing compatibility threads and rebuilding the packages manually.

- Built the image with **_cmake_** + **_make_** → This taught me that even “Python demos” on Jetson often require native compilation because they wrap optimized CUDA/TensorRT libraries in C++. “Python entrypoints” aren’t just scripts; they depend on compiled GPU-accelerated backends.
- **_sudo make install_** **_\+ sudo ldconfig_** → Jetson places libraries in /usr/local/lib. Without running ldconfig, the OS wouldn’t know where to find those shared objects (.so files), and your Python imports (import jetson_inference) would fail. ldconfig is how you “teach” Linux about new GPU libraries you just built.

Eventually, I realized → sometimes it’s faster to pivot than debug legacy. The tutorials still referenced Caffe-based models (.caffemodel), but TensorRT 10.3 (JetPack 6.x) dropped support for Caffe entirely.

At the time, I glossed over it, _just another package issue I'd work around (or so I thought)._ After looking back and realizing how much time I spent debugging it, I thought it warranted a deeper look. So I decided to ask, “What is a caffemodel?”

A **caffemodel** is a serialized neural network format from the old Caffe deep learning framework (popular ~2014–2017). Caffe gained traction because it made computer vision tasks like image classification and segmentation fast and accessible at the time. As **TensorFlow** and **PyTorch** rose, Caffe usage waned; its last official release was **2017**, and it’s been effectively end-of-support since around **2018** (with **Caffe2** later folded into PyTorch). **\[**[GitHub: BVLC/caffe](https://github.com/BVLC/caffe)**\]**.

Caffe relies on two primary files, the .prototxt and .caffemodel

- A .prototxt file defines the architecture of a neural network—layers, connections, parameters—using the Protocol Buffers format. It’s essentially the blueprint of the model.
- A .caffemodel file contains the trained weights (the actual learned parameters). It's the result of the training process.

In practice (and on Jetson’s jetson-inference), the process happens like this: The system reads the .prototxt to build the computation graph. It then loads the .caffemodel to fill in the weights. Finally, TensorRT takes over to optimize the network for GPU inference—if supported. [\[Medium: Caffe, Onnx, Tensorflow and Pytorch frameworks\]](https://medium.com/%40abhishekjainindore24/caffe-onnx-tensorflow-and-pytorch-frameworks-5c01c7045644)

![Caffe model flow]({{ page.images_path }}/caffe-flow.png)

As amazing as this all sounds, after digging through the NVIDIA forums, the bottom line was: you can’t force TensorRT to run obsolete models anymore. I'm sure there is a way you can (i.e. using compatible or legacy packages). I saw threads about deleting .caffemodel + .prototxt files, but I had hit my debugging breaking point. 😆 At some point I acknowledged my limited (or lack of) foundational understanding of these packages and this technology. It was time for a new direction.

**Key-Takeaway 2**: don’t waste time resurrecting deprecated frameworks — lean into what you know and the toolchain NVIDIA supports today (PyTorch + TensorRT via ONNX). Fail often, fail fast still holds true in many technical environments.

Side note:

_In my forum scrolling I also learned that the primary maintainer of the repo had stepped down at NVIDIA the week prior. This made moving through some parts of the project difficult because it was unclear what would be supported and what would remain broken. Just goes to show how fragile and niche these projects are. See the forum here_ [**\[NVIDIA Forum\]**  
](https://forums.developer.nvidia.com/t/jetson-ai-lab-dead/341702/9)_(Shoutout to dusty-nv and the team, sounds like they put a lot of work into this. It's people like them that make this stuff possible for people like me.)_

### **Pt 3: The Pivot** 🔁

_‘Trying to solve the caffemodels error was like dusting off an old VHS tape, only to realize your TV doesn’t even have a VHS player anymore. At some point, it’s easier to stream the movie than fight with adapters.’_

Instead of burning more time fixing outdated Caffe demos, I moved from “canned demos” to something more forward-looking and relevant, like text-generation-webui containers and n8n workflows. I switched to PyTorch/ONNX models (like ResNet, LLaMA, etc.) that are actually supported in JetPack 6.x.

**Text Generation** ⚡️

I pulled the nano llm image through jetson-containers and ran models locally with GPU acceleration. I'll probably write more about that at some point, specifically which ones I ran and how I looked at performance on the nano, but for now I just wanted to point out that this basic test gave me hands-on confirmation of what I had suspected — Jetson Nano isn’t built to outpace a dev laptop or cloud GPUs on raw performance. It's an **edge sandbox** for running inference where sending data back and forth to the cloud isn’t practical (think robotics, IoT, or low-latency automation). The key learning for this wasn’t about “speed tests” — it was about understanding the **fit for purpose** use case. Jetson is less about competing with other hardware, more about letting you prototype and test \_deployment-ready AI\* at the edge.

**Automatio**(_n8n_)

At this point, my appetite for LLM text generation was satisfied, and I wanted to move on to something else. With my experience as a data engineer, I’ve always been drawn to any and all automation tools. n8n was one of those tools that I had heard about, but had never found a good opportunity to POC. Sure enough, it happened to be one of the suggested demos for the Jetson 🎉 I just needed to find a good use case for it, and to start, I needed to better understand what it was.

n8n is a **fair-code (SUL) workflow automation tool** that **combines AI features with business process automation**. It’s best compared to **Zapier/Make** (low-code integrations) and **Apache NiFi/Node-RED** (visual dataflows). It overlaps with **Airflow/Luigi/Step Functions** on orchestration/scheduling, but those are code-centric orchestrators rather than integration platforms.

As someone who uses **Apache Airflow** for data engineering, the contrast was clear:

- **Airflow** excels at orchestrating large, complex data pipelines with dependencies and SLAs.
- **n8n** shines when you want low-code, reactive automations that respond to events (APIs, webhooks, bots).

Being a frequent Discord user, I was aware of the API access available for bots, so I quickly began brainstorming POC ideas to test n8n. Honestly, I found it challenging to choose which idea to pursue, but ultimately, I settled on one that would allow me to complete the POC as quickly as possible. In the end, I decided to build a quick workflow that pulls a random Chuck Norris joke from an **\[**[API](http://api.chucknorris.io)**\]** and sends it to my Discord server _every minute_ through a webhook. 🥋 🧔 riveting stuff 😆

![n8n workflow UI]({{ page.images_path }}/n8n-flow.png)

![POC image]({{ page.images_path }}/poc-image.png)

It sounds simple, _and it really is. n8n is pretty impressive_.  It showcased how n8n makes **event-driven automation** dead easy. What would take a handful of scripts, scheduling, and error handling in plain code, I stitched together visually in under an hour.

n8n differentiates itself from others with its ability to tie in LLM workflows. It offers rich support for LLMs with dedicated nodes for OpenAI, Anthropic, Ollama, Azure, Google Gemini, and more. You can also call any LLM or custom endpoint using the raw HTTP Request node. Unlike Zapier, which offers limited prompt templating and cloud-only options, n8n gives you full control via LangChain integration and prompt construction. See here for an interesting article comparing it to Zapier → [n8n-vs-zapier](https://blog.promptlayer.com/n8n-vs-zapier/)

**Key-Takeaway 3**: LLM text generation served up by 3rd party providers, in almost all cases, will be faster than your small dev edge device. However, it won't have the same fine-tuning or security capabilities of locally run models. In addition to running LLMs, the Jetson also offers workflow automation integration. n8n has emerged as an early entrant to the LLM workflow automation space, providing an easily deployable low-code and no-code solution for workflow automations.

### **Final Thoughts**

I’ve learned a lot in this process, and had a lot of fun doing it. Working through bugs and trying different technologies has opened my eyes to new opportunities and projects I'd like to try. The world of edge computing, and specifically its relation to AI, is evolving rapidly. Take caffe, for example, the concepts are hard to grasp and even harder to master, and by the time you’ve mastered it, it’s been replaced by PyTorch and TensorFlow.

As these edge devices improve alongside AI, I can see ‘plug and play’ improving to a point that these devices would be accessible to everyday consumers. Eventually, many of the difficult concepts we struggle with will be abstracted away by superior hardware/technology. Until then, it will continue to be used for niche enthusiast projects or production-grade edge devices.

So even though I didn’t get image detection to work on my first try, and I only ended up creating a Chuck Norris joke bot, it was still completely ‘worth’ as some would say 😆. In the end, it’s important to realize that not knowing everything is okay. Fail often, try new things, use the knowledge you possess to discover the things that actually do work. Find joy in the small wins and have faith that each little failure is building to something much larger.

**_What’s next?_** Well, for one, I do want to get image detection working, but also, I’m not super impressed with my bot. That’s all it is. I could have done that through scripts. I want to deploy a Discord workflow on n8n that leverages the LLM n8n workflow.

👉 If you’ve experimented with **Jetson, n8n, or Discord bots**, I’d love to hear what you built!

💡 **Tips**

- The Jetson is best used as a **playground for edge AI + automation ideas**.
- Community blogs + forums (JetsonHacks, NVIDIA) saved me from dead ends more than once.
- **Chromium broke entirely** on Jetson (packaging issue). Thanks to JetsonHacks + community patches, I got it fixed. Another lesson in the value of forums and shared knowledge.
