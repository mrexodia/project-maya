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
