# Inkling 2 Mini on an 8 GB card: test build

[Project Maya](https://github.com/mw00/project-maya) v1.1.75 plus
[#1129](https://github.com/mw00/project-maya/pull/1129), [#1130](https://github.com/mw00/project-maya/pull/1130) and
[#1144](https://github.com/mw00/project-maya/pull/1144). We need speed numbers from a real 8 GB NVIDIA card.

**You need:** an 8 GB NVIDIA card (RTX 20 series or newer), 16 GB of RAM or more, 25 GB of disk, Linux or Windows.
Windows also needs CUDA 12.8 and Visual Studio 2022 Build Tools ([docs/WINDOWS.md](docs/WINDOWS.md)).

## 1. Download the model

unsloth's `UD-Q4_K_XL` quant (17.6 GB), at the revision Maya checks against:

```sh
git clone -b inkling-8gb-test https://github.com/mrexodia/project-maya.git
cd project-maya
hf download unsloth/Inkling-2-Mini-GGUF Inkling-2-Mini-UD-Q4_K_XL.gguf \
    --revision ae9327c5fbc7333116ba32eee78bb5d2bbefae4f --local-dir models/Inkling-2-Mini
```

## 2. Set up and start

Builds the engine (20-40 minutes), then starts the model at `http://127.0.0.1:8080`. Enter accepts the defaults.

```sh
./setup.sh --model inkling-2-mini --model-dir models/Inkling-2-Mini
```

Windows: `START-MAYA.bat --model inkling-2-mini --model-dir models\Inkling-2-Mini`

Don't install the update the page offers: it replaces this branch.

## 3. Benchmark and report

Stop Maya (Ctrl+C), close other programs that use the GPU, then:

```sh
./maya.sh --bench
./maya.sh --report
```

Windows: `START-MAYA.bat --bench`, then `START-MAYA.bat --report`.

Send back `maya-bench-results.txt`, `maya-report.txt`, and your GPU, its PCIe link (e.g. 4.0 x8), CPU and RAM.

## 4. Second run: less VRAM kept free

On a card that cannot hold every expert, the engine keeps part of the VRAM free and fills the rest with the experts
used most often. `IK2_ROOM_MB` sets how much stays free (default 1024 MB). Less free VRAM means more experts on the
card and fewer copies from RAM. Too little can make Windows move GPU memory to RAM, which is much slower.

Rename the two result files from step 3, then set up again with the setting (nothing is downloaded or built again)
and repeat step 3:

```sh
./maya.sh --setup --model inkling-2-mini --env IK2_ROOM_MB=256
```

Windows: `START-MAYA.bat --setup --model inkling-2-mini --env IK2_ROOM_MB=256`

What we measured on an RTX 3090 limited to an 8 GB card's free VRAM (Windows, an earlier build that kept 2 GiB free
by default, 16K context):

| VRAM kept free | Experts on the card | Prompt, 4.2K tokens | Prompt, 12.6K tokens | Answers, prose | Answers, code |
|---|---|---|---|---|---|
| 2 GiB (that build's default) | about 2.0 GB | 634 tok/s | 730 tok/s | 22 tok/s | 21 tok/s |
| 256 MiB | about 3.8 GB | 859 tok/s | 1,079 tok/s | 36 tok/s | 31 tok/s |

This build keeps 1 GiB free by default, so expect a smaller gain from 256 than in the table. A real 8 GB card has
less memory bandwidth than a 3090, so expect lower numbers overall.
