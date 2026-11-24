# Inkling 2 Mini on an 8 GB card: test build

This branch is [Project Maya](https://github.com/mw00/project-maya) v1.1.75 with three pending changes
([#1129](https://github.com/mw00/project-maya/pull/1129), [#1130](https://github.com/mw00/project-maya/pull/1130),
[#1144](https://github.com/mw00/project-maya/pull/1144)). It needs speed numbers from a real 8 GB NVIDIA card.
So far 8 GB has only been simulated on an RTX 3090, which has more memory bandwidth than an 8 GB card.

The whole test takes about an hour, most of it the download and the build.

## What you need

- **GPU:** one NVIDIA card with 8 GB, RTX 20 series or newer (GTX 10 series and older are not supported).
- **RAM:** at least 16 GB; 32 GB is better. The model is about 18 GB, and whatever does not fit on the card is kept
  in RAM.
- **Disk:** about 25 GB free.
- **OS:** Linux or Windows 10 / 11.
  - Linux: the NVIDIA driver, the CUDA toolkit, `git`, `cmake`, a C++ compiler and Python 3.10 or newer.
  - Windows: the NVIDIA driver, [CUDA Toolkit 12.8](https://developer.nvidia.com/cuda-12-8-0-download-archive),
    [Visual Studio 2022 Build Tools](https://aka.ms/vs/17/release/vs_BuildTools.exe) with "Desktop development with
    C++", `git`, and 64-bit Python 3.10 or newer (`winget install -e --id Python.Python.3.12 --scope user`). Details:
    [docs/WINDOWS.md](docs/WINDOWS.md).

## The model

[Inkling 2 Mini](https://huggingface.co/thinkingmachines/Inkling-2-Mini) (26B parameters, a mixture of experts), as
unsloth's `UD-Q4_K_XL` GGUF (17.6 GB). Its setup id is `inkling-2-mini`; the setup downloads it with its vision file
and drafter.

On an 8 GB card the engine keeps the experts used most often on the card and the rest in RAM, and copies a missing
expert to the card when a token needs it. The setup works this out by itself: no settings are needed for a first
run. The default context is 32,768 tokens.

## Steps

**1. Get this branch.**

```sh
git clone -b inkling-8gb-test https://github.com/mrexodia/project-maya.git
cd project-maya
```

**2. Set up the model.** This shows the downloads, then builds the engine (20-40 minutes) and starts the model.
Press Enter to accept the recommended answers.

- Linux: `./setup.sh --model inkling-2-mini`
- Windows: `START-MAYA.bat --model inkling-2-mini`

The first start takes a few minutes while the engine loads the experts into RAM.

**3. Try it.** Open `http://127.0.0.1:8080`, ask a short question, then paste a long text or code file and ask
about it. Note how long the long prompt takes before the answer starts.

If the page offers an update, **do not install it**: it replaces this branch with the official release.

**4. Run the benchmark.** Stop Maya first (Ctrl+C in its window) and close other programs that use the GPU (games,
browsers with video).

- Linux: `./maya.sh --bench`
- Windows: `START-MAYA.bat --bench`

It takes a few minutes: a warm-up, three 256-token answers, and 2K and 8K-token prompts three times each. It prints
one result line and writes it to `maya-bench-results.txt`.

**5. Write the report.**

- Linux: `./maya.sh --report`
- Windows: `START-MAYA.bat --report`

This writes `maya-report.txt` with your hardware, the setup and the engine's log. Nothing is sent anywhere.

**6. Optional: the CPU helping with experts.** `IK2_LANE=1` lets the CPU compute some of the experts that are not on
the card, instead of copying all of them over. Whether that is faster depends on the CPU and the PCIe link. To test
it, set it up again with the setting (nothing is downloaded or built again), then repeat steps 4 and 5:

- Linux: `./maya.sh --setup --model inkling-2-mini --env IK2_LANE=1`
- Windows: `START-MAYA.bat --setup --model inkling-2-mini --env IK2_LANE=1`

Rename `maya-bench-results.txt` and `maya-report.txt` from the first run before this, so they are not overwritten.

## What to send back

- `maya-bench-results.txt` and `maya-report.txt` (from both runs, if you did step 6)
- your GPU, its PCIe link (for example "PCIe 4.0 x8"; laptops are often x8 or x4), CPU, RAM size and OS
- how the chat felt: the speed of answers, the wait before a long prompt's answer starts, anything that went wrong

## For comparison

| Setup | Prompt reading | Answers |
|---|---|---|
| RTX 3090 limited to an 8 GB card's free VRAM, Windows, an earlier build (16K context, a 12.6K-token prompt) | 730 tok/s | 21-22 tok/s |
| V100 limited to 8 GB, Linux, upstream's measurement (32K context, a 2K-token prompt) | 927 tok/s | 15.0 tok/s |
| RTX 3090, every expert on the card (24 GB) | about 2,500 tok/s | 116 tok/s (prose) to 205 tok/s (code) |

## If something goes wrong

- **The build stops:** [docs/SETUP.md](docs/SETUP.md#if-the-build-fails) has the fix for the common errors.
- **The setup refuses to start:** it prints one line saying why (usually too little RAM for the model). Send that
  line.
- **Windows: answers suddenly several times slower:** the card may be too full, so Windows moves GPU memory to RAM.
  Close other programs that use the GPU, start again, and mention it in your report.
- **Anything else:** run step 5 right after it happens and send `maya-report.txt`. The engine's own log is
  `maya-inkling-2-mini.log` (or a similar `maya-*.log`) in this folder.

Everything else about Maya: the official [README](https://github.com/mw00/project-maya#readme) and
[docs/INKLING2.md](docs/INKLING2.md).
