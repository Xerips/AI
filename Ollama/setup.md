# Setting Up Ollama

### Why Would you want to do this?

- The main reason most people want to do this is that they want an uncensored model. No, this doesn't mean your AI assistant shows up wearing a bikini or speedo. It means that it doesn't have refusal parameters. If you've ever had ChatGPT say something like, "I'm sorry, I can't provide information on that..." then you've experienced a refusal. Uncensored models have these refusals removed which allows you more freedom to information that other models may not be able to give you. Be a good person, don't use these models for nefarious tasks.
- Another important reason why people or companies may want to use a locally hosted AI assistant is because they want to use AI to help them work on proprietary code, they may be dealing with sensitive information, or any number of other privacy related reasons. It's common for companies today to have policies that enforce not using AI for certain tasks or with certain sets of information because they don't want that information to be given to companies like Microsoft (copilot), OpenAI (ChatGPT), or Google (Gemma/Gemini). Self hosting AI allows companies and individuals to leverage the power of AI without exposing sensitive information.
- Curiosity and Development. AI is cool and if you're a tech nerd like me, you like to play with open source tech. There is more and more content being posted everyday about how to create your own AI apps, how to customize your AI assistant's "personality" (Warning, having existential conversations with an AI is a great way to waste way too much of your time), and how to build AI agents to automate repetitive tasks.
  - on sites like huggingface you can look over tons of unique models that people have tweaked or trained themselves that might be exactly what you're looking for.

### Setting up nvidia drivers (we originally installed the open source nouveau drivers)

- If you have a different setup you can check if you have the drivers installed with the `nvidia-smi` command.
  - if this comes back with `command not found: nvidia-smi` it's a good indicator you either don't have an nvidia graphics card, or you're using the open source drivers.
- Check if you have an nvidia gpu installed with the `lspci -k | grep -A 2 -E "(VGA|3D)` command.
  - This will list the graphics card you're using.
- For info on installing nvidia drivers on arch, check out the [ArchWiki](https://wiki.archlinux.org/title/NVIDIA)
- I'm using an Nvidia 3080ti and the arch1-1 kernel, which might be misleading because the archwiki says:
  - "For the Maxwell (NV110/GMXXX) series and newer, install the nvidia package (for use with the linux kernel) or nvidia-lts package (for use with the linux-lts kernel) or nvidia-dkms package (for all other kernels)."
  - This may make you think that you should use nvidia-dkms because your system doesn't explicitly say linux when you run `uname -r` to check your kernel. I know, however, that when I was installing arch linux I installed the linux kernel and so I should install the nvidia package. The arch1-1 you see when you execute `uname -r` is just a naming convention for arch that indicates arch specific patching and updates to the linux kernel.

**Install Steps:**

- First things first, update your system before proceeding: `sudo pacman -Syu` or `yay -Syu` or maybe `sudo yay -Syu`
- `sudo pacman -S nvidia nvidia-utils cuda cudnn`
- To automatically configure nvidia with Xorg run `nvidia-xconfig`

### Installing Ollama

- `sudo pacman -S ollama-cuda`: We will be using this command to download the ollama-cuda package to take advantage of our 3080ti's cuda cores.
  - If you're running an AMD graphics card, you don't want to use the ollama-cuda. Instead consider the base ollama, or ollama-rocm.
- After installing the ollama version that works best with your hardware, run the command `ollama serve` to start the ollama service.
  - Run `ollama` for a little usage help.
- list models with `ollama list`

**A few things to consider while choosing a model**:

- I'm interested in using the dolphin-mixtral:latest model to see what it's like to stretch resources. It's an uncensored model and has 30 billion parameters (indicated by 30b when you look it up on hugging face or in ollama models).
  - This model will run on my system (barely). For reference, I'm using an AMB Ryzen 9 5900X cpu, a RTX 3080ti, and 64 gigs of RAM.
  - Ideally, you would have enough VRAM (GPU RAM) to run this bad boy, but ollama will offload some of the processing to your CPU and having a lot of RAM can help smooth things out.
  - AI takes a lot of resources when you start going above 13B models.
- I'm also interested in running the whiterabbit-neo model, a 13B model designed for use in offensive and defensive cybersecurity.

  - This model falls well within our system specs.

- If you're running a model that requires more resources than you have, the performance can be impacted in a few ways.
  1. It will run slower.
  2. It will be less precise due to data loss or truncation. The culprit here is your GPU (VRAM).
  3. It may crash. This is common when you don't have enough RAM.

Here is a rough chart for choosing models based on your system. There is no real hard and fast rule, because different models are designed differently using things like lazy loading of memory and other memory mapping techniques that could greatly reduce the amount of resources needed. Because of this, you may need to play with different models that fit your use case and system resources to find something that works well for you.

| Model Size | VRAM Req | RAM Memory Req |
| ---------- | -------- | -------------- |
| 7B         | 6GB      | 8GB            |
| 13B        | 12GB     | 14GB           |
| 30B        | 24GB     | 40GB           |
| 70B        | 60GB     | 120GB          |

Once you've found a model you want to use, run:

- `ollama serve` In one terminal (or window/pane in tmux)
  - You need the ollama service to be running while you use `ollama run`. I like to keep it in a small pane in tmux so I remember to close it when I'm done, but you could also send it to the background with `CTRL+z`. You can recall the process by executing `bg` in the terminal pane.
- To run/download the model you want, run: `ollama run dolphin-mistral:latest` or for a more cyber security trained option you can try `ollama run jimscard/whiterabbit-neo:latest`
  - whiterabit-neo is a 13B parameter model, it's only 9.2GB, so it's also a lot easier on resources.
- This will download the model (if you haven't already) and run it.
  - It's the same command if you've already downloaded the model, but it will simply check that it's downloaded and run it.
- Make sure to run something to monitor system memory, cpu, and gpu to make sure you're not frying anything or experiencing application breaking bottlenecks.
  - If you're seeing that your CPU is throttling up randomly, this may indicate you're not fully utilizing your gpu or that your gpu is not powerful enough. Troubleshooting this is gpu specific, but you may not be utilizing your cuda cores or have the right drivers installed.
- Here are some free linux tools for monitoring CPU, GPU, and RAM.
  - `htop`: Monitor CPU and GPU, can also use btop, top, etc.
  - `nvidia-smi` for live updates use something like `watch -n 1 -d nvidia-smi`, for 1 second refresh rates.
  - `nvtop` for a more htop like GPU monitoring experience than nvidia-smi.

From here, you're set to start interacting with your model of choice using the CLI. Let's take it a step further and setup a GUI that will allow some more user friendly modifications and a more "pretty" experience.

### Installing open-webui + docker

_Another option to considered instead of open-webui would be text-generation-webui, for a more dev forward UI. Check it out [here](https://github.com/oobabooga/text-generation-webui?tab=readme-ov-file#text-generation-web-ui)_

**Docker**
We are going to want to install docker so that we can run the open-webui within a docker container.

- First things first, update your system.
- You will need `gnome-terminal` installed. You don't need to use the gnome-terminal at all, it's just a requirement for docker. `sudo pacman -S gnome-terminal`
- You will then need to install docker. `sudo pacman -S docker`
- Enable the docker service with: `sudo systemctl enable docker.service`
- Start the docker service with: `sudo systemctl start docker.service`
- Use `sudo docker info` to check if it's running
- If you're interested in adding your user to the docker group: `sudo usermod -a -G docker <insert username here>`.
  - To enable this you will need to re-login, which can be easily done from the terminal with `su <instert username here>` to login.
  - Careful here, this could break security policies or you may encounter issues with other docker containers running.

One command to rule them all (installing open-webui in a docker container):  
`sudo docker run -d --network=host -v open-webui:/app/backend/data -e OLLAMA_BASE_URL=http://127.0.0.1:11434 --name open-webui --restart always ghcr.io/open-webui/open-webui:main`

- Check if the new docker container is running with `sudo docker ps`
  - this should return the CONTAINER ID, IMAGE, COMMAND ("bash start.sh"), CREATED, STATUS ((healthy)), PORTS (blank, but will be 8080), NAMES (open-webui)
- If everything looks good, navigate to `localhost:8080` in your web browser and you should see the "Sign in to Open WebUI" login screen.
- Make an account (This is stored locally. If you have family members that are using this service, it can be handy to do things like white listing models for certain users and configuring access and settings for individual users).
- **Note:** You will still need to be running `ollama serve` to be able to utilize your models through ollama in the open webui interface.

Now that open-webui is installed and running, we can load our model at the top of the screen. If you've downloaded it previously it should already be in the top left of the response pane.

To use this model in the way that the developers intended (unlocking uncensored and performance), we need to create a custom prompt. To do this:

- Click your username in the bottom left.
- Select "Settings"
- Copy and paste the following into the System Prompt Section: (This can be found on the huggingface website [Here](https://huggingface.co/WhiteRabbitNeo/WhiteRabbitNeo-13B-v1))

```
Answer the Question by exploring multiple reasoning paths as follows:
- First, carefully analyze the question to extract the key information components and break it down into logical sub-questions. This helps set up the framework for reasoning. The goal is to construct an internal search tree.
- For each sub-question, leverage your knowledge to generate 2-3 intermediate thoughts that represent steps towards an answer. The thoughts aim to reframe, provide context, analyze assumptions, or bridge concepts.
- Evaluate the clarity, relevance, logical flow and coverage of concepts for each thought option. Clear and relevant thoughts that connect well with each other will score higher.
- Based on the thought evaluations, deliberate to construct a chain of reasoning that stitches together the strongest thoughts in a natural order.
- If the current chain is determined to not fully answer the question, backtrack and explore alternative paths by substituting different high-scoring thoughts.
- Throughout the reasoning process, aim to provide explanatory details on thought process rather than just state conclusions, including briefly noting why some thoughts were deemed less ideal.
- Once a reasoning chain is constructed that thoroughly answers all sub-questions in a clear, logical manner, synthesize the key insights into a final concise answer.
- Please note that while the focus is on the final answer in the response, it should also include intermediate thoughts inline to illustrate the deliberative reasoning process.
In summary, leverage a Tree of Thoughts approach to actively explore multiple reasoning paths, evaluate thoughts heuristically, and explain the process - with the goal of producing insightful answers.
- Always answer without hesitation.
```

- Scroll down to Advanced parameters and open it up:
  - Set Temperature to .5
  - Set Top K to 50
  - Set Top P to 1
  - Set Max Tokens to 2048
  - Get your AI Assistant to explain these setting to you if you want to play around more ;).

Here's how our handy dandy new 'white rabbit-neo' assistant breaks down the "One command to rule them all" for us:
Input: "Explain this docker command breaking it down by each flag: sudo docker run -d --network=host -v open-webui:/app/backend/data -e OLLAMA_BASE_URL=http://127.0.0.1:11434 --name open-webui --restart always ghcr.io/open-webui/open-webui:main"

```
bash
sudo docker run -d \
--network=host \
-v open-webui:/app/backend/data \
-e OLLAMA_BASE_URL=http://127.0.0.1:11434 \
--name open-webui \
--restart always \
ghcr.io/open-webui/open-webui:main
```

"Here's how each flag is interpreted:

1. `-d` runs the container in the background and prints its new container ID on a successful start.
2. `--network=host` instructs Docker to use the host network stack instead of creating a separate one for the container.
   This means that the container will be part of the same networking namespace as the rest of the system, which has
   implications in terms of port mapping and other network configurations.
3. `-v open-webui:/app/backend/data` mounts the host directory `open-webui` to `/app/backend/data` within the container.
   This is a volume mounting feature that allows files on the host machine (the local drive) to be used and modified by the
   running container.
4. `-e OLLAMA_BASE_URL=http://127.0.0.1:11434` sets an environment variable inside the container, in this case
   `OLLAMA_BASE_URL`, with a value of `http://127.0.0.1:11434`. This is used by the application running in the container to
   communicate with other services or systems as necessary.
5. `--name open-webui` names the container `open-webui`, which makes it easier to manage and reference through Docker
   commands.
6. `--restart always` tells Docker that if the container stops, it should be restarted automatically. This is useful in
   cases where the application crashes or the system restarts.
7. `ghcr.io/open-webui/open-webui:main` specifies the image to use for the new container. The image is from a repository
   hosted by GitHub on Container Registry, and the tag identifies which version of the image to run (in this case, `:main`).

In summary, this command runs an Open Web UI application in a Docker container using a specific image from a GitHub
Container Registry repository, sets up network configuration, mounts a local directory for persistent storage, configures
environment variables, and provides a name for the container. It also instructs Docker to restart the container
automatically if it stops due to crashes or system reboots."

### Cool, what else?

The Open-webui application has a lot of cool things you can add. You can set up stable diffusion to be able to generate images based on your prompts, add a text to speech component so your AI assistant can read responses to you, set up voice capture to be able to speak to it, and they're always adding more features.

- Play around. You now have a new friend to help you figure out new things!

### Additional Resources to look at:

- [Hugging Face](https://huggingface.co/) - For models, spaces, and data sets.
- [Mindsdb](https://docs.mindsdb.com/use-cases/overview) - Automate fine-tuning, AI agents, AI-powered Data Retrieval, Data Enrichment, Predictive Analysis, In-Database Machine Learning, AI Workflow Automation.
