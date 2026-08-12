
This course will ask you to run quite large experiments on LLMs. For this you
need to make some decisions on compute. The case studies (assignments) might
give pointers on how to best run that specific experiment, but to help you get
prepared we also give you a general overview. The goal of the document is not
guide you through the assignments, only to make the assignments less
frustrating. You are therefore invited to share any problems and solutions you
have found with the teaching team. You can do this through opening a pull
request in which you supplement this document.

By far the simplest case is if you have a computer with a reasonably good
GPU[^1](16 GB VRAM) or a MacBook with an M series chip and at least 16gb of
RAM. In this case you should be able to do the first 3 case studies without any
problems on the hardware front, though you will be limited to small models. For
the last case study this is also possible, but it will require some planning as
it takes quite a while and does not allow for easy parallel requests. All these
experiments do allow you to use your computer as a fancy radiator, thereby
offsetting some of your electricity use through lowering your heating bill.

In case neither you nor your partner has adequate hardware, we have a few
suggestions. These can be split into:
1. University Facilities
2. Commericial Options
3. Patience

The university has multiple facilities for computing. All of them run some
version of Linux (Rocky or Ubuntu).  For a thorough overview of all facilities
and the rules on using them consult rel.liacs.nl. These computers are not
reserved for you, and there is no way to 'claim' one. It is up to you to make
sure you have enough time to secure yourself a spot. There are many machines in
the Gorlaeus building that are equipped with a RTX 4060 (8 GB of VRAM), located
in DM.0.09, DM.0.13, DM.0.17, DM.0.21. Note these only have 8 GB of VRAM,
meaning you are limited to very small models. 

If you are not comfortable using `ssh` this is likely the easiest way to get
yourself setup with a reasonably performant computer. Note that these computers
also allow you to use `ssh`, so you can continue/monitor your work remotely.

There are also GPU nodes available on the REL cluster, 3 of theres are available
to students, for a total of 10 GPUs. These you can only access through `ssh`.
If you decide to use them please make sure to read the storage rules, as you
will easily go over the storage limit if you accidentally work a non-local
disk. These computers do have better GPUs than those in the DM.0.* rooms,
thought tend to be in use more.

As for commercial options, there are many providers, such as google collab. In
principle, it does not matter which one you use, and cost should be reasonably
low.  Again it is important to start on time. Also mind the cost, some
assignment will burn through a lot of tokens, be careful not rack up a large
bill (e.g. trying to use Claude Opus or ChatGPT Sol will easily cost your 50-200 euros).

Finally, though slow and painful, a good CPU can get you through most of the
course. Case study 1 will take about 45 hours of compute time on a CPU, at
least as tested on and AMD Ryzen 9 3950X 16-Core Processor. For reference, on
an RTX 3090 this will be on the order of 4-8 hours. The other case study do not
train models, and therefore benefit less from having a GPU, the mainly push
your memory bandwidth, this again makes MacBook with a large amount of RAM
excellent local hardware. This should however be considered a last resort,
unfortunately because of the long times involved you do not have the luxury to
wait until the last moment before making this decision.


### Multi-GPU Devices 

If you end up working on a work station with multiple GPUs, some, but hopefully
not all, GPUs might be busy. In order to let `pytorch` know which GPU is 'safe'
to use, you should use:

```bash
export CUDA_VISIBLE_DEVICES=$GPU_NUMBER
```

Where `$NUM` is the number associated with the GPU you want to use. To get an
overview you can use either `nvida-smi` or `btop`.

## On Package management

Use `uv`. Other options will work too, but are generally worse, especially when running
your code on REL, as you will not be able to make use of the cache. You'll
notice that the assignments all have different ways of managing packages as
they are written by different people at different times. `uv` will be able to
deal with all this, while being faster and more ergonomic. If you have not used
`uv`, this might be a good time to try it.

Computation: runs plenty fast on CPU


# Case Study 2 

BALROG uses a `setup.py` file, which expects an older python version. With `uv`
you can create a virtual environment with the right python version.

```bash
uv venv --python 3.11 
source .venv/bin/activate 
```

After which you can simply install using `uv`

```bash
uv pip install -e . 
```

If this fails it might  be the case that you are missing some build
tools, you should read the errors as they will likely tell you, but these could
include `bison`, `flex` and `gcc`.

On the REL servers just creating a venv and running 

```bash
uv pip install -e . 
```

will not work. As the standard python version will be python 3.9, but balrog
(the textworld dependency specifically) requires 3.10

If you end up using a cloud provider you might need to make some changes to
`balrog/client.py`. If the provider is compatible with either openAI, Anthropic
or Google's api's it will be easiest to simply add a case where you manually
provide your API-key. For example, when using `openrouter`, you can add to the
`openai` wrapper as follows:

```python
if self.client_name.lower() == "vllm": 
    self.client = OpenAI(api_key="EMPTY", base_url=self.base_url) 
# Add a new case for openrouter
elif self.client_name.lower() == "openrouter": 
    self.client = OpenAI(api_key=os.environ["OPENROUTER_API_KEY"], base_url=self.base_url)
```

We cannot provide a complete overview for all providers, it is therefore your
responsibility to pick one and figure out if it is compatible with others. If
you decide to use a provider which does not happen to be compatible with any of
the aforementioned you will have to write your own wrapper. 

# Case Study 3

Only a requirements file is supplied, but `uv` can install this using: 
```bash
uv add -r requirements.txt 
```


# Case Study 4 There is a conflict in
No special remarks

[^1]: Nvdia is better supported, but AMD should be fine. As far as I know Intel
    GPUs (Arc) are not supported by `pytorch`, the same goes for 'NPUs' your
    computer might come with. 
