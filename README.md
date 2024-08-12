## AutoRT: the Next Generation of Antares.

***AutoRT for Device Runtime:***

AutoRT is a compiler solution that helps runtime users to invent, benchmark and optimize operators for Pytorch using your own accelerators:
- AutoRT can be as a [benchmark utility](#--playground-1---benchmark-your-windows-device) for device performance testing and profiling.
- AutoRT can also generate Pytorch2 of your device to accelerate standard [Pytorch applications](#quick-test-2-mnist-training-by-pytorch2-using-windows-directx) (e.g. DirectX).
- Additionally, AutoRT futher helps to construct [custom defined](#quick-test-1-create-custom-operator-of-your-own-in-pytorch-2) / fused operators that are beyond the built-in functions of Pytorch.
- ***AutoRT for Windows DirectX 12 / Linux CUDA*** has experimental version [released](#--quick-installation-of-autort).
- Click [here](https://github.com/microsoft/antares/issues/new) to suggest more platforms (e.g. Pytorch2 for Windows ROCm / OpenCL / SYCL / Apple Metal / ..) you would like AutoRT to support in the follow-up releases.

#### Archtecture of AutoRT as a Backend for Pytorch 2.0:
<p align="center">
  <img src="AutoRT4Torch.svg" data-canonical-src="AutoRT4Torch.svg" width="650" height="230" />
</p>

#### Workflow of Custom Operations from Antares IR to Different Backends:
<p align="center">
  <img src="AutoRT-opt.svg" data-canonical-src="AutoRT-opt.svg" width="650" height="120" />
</p>


## - Quick Installation of AutoRT:

#### Installation

| Platform | OS Requirement | Python Requirement | Download Link |
| --- | --- | --- | --- |
| DirectX 12 (x86_64) | Windows >= 10 / Microsoft XBox | [Python3.12](https://www.python.org/ftp/python/3.12.0/python-3.12.0-amd64.exe) (Windows) | python3.12 -m pip install https://github.com/microsoft/antares/releases/download/v0.9.6/autort-0.9.6.3+directx.win-cp312-cp312-win_amd64.whl |
| Vulkan 1.3 (x86_64) | Ubuntu >= 18.04  | Python3.12 (Linux) | python3.12 -m pip install https://github.com/microsoft/antares/releases/download/v0.9.6/autort-0.9.6.3+vulkan.linux-cp312-cp312-manylinux1_x86_64.whl |
| CUDA >= 11.0 (x86_64) | Windows >= 10 / Ubuntu >= 18.04 | Python 3.10/3.12 | python3 -m pip install https://github.com/microsoft/antares/releases/download/v0.9.6/autort-0.9.6.3+cuda.zip |
| CUDA >= 12.0 (aarch64) | Ubuntu >= 22.04 | Python 3.10/3.12 | python3 -m pip install https://github.com/microsoft/antares/releases/download/v0.9.6/autort-0.9.6.3+cuda.zip |
| .. | .. | .. | .. (More coming soon) .. |

For CUDA, here are several Ubuntu >= 18.04 equivalent containers below:
 * **Docker Image:** nvidia/cuda:12.0.1-cudnn8-devel-ubuntu18.04
 * **Docker Image:** nvidia/cuda:11.8.0-cudnn8-devel-ubuntu20.04
 * **Docker Image:** nvidia/cuda:12.0.1-cudnn8-devel-ubuntu20.04
 * **Docker Image:** nvidia/cuda:12.1.0-cudnn8-devel-ubuntu22.04
 * ..

## - Playground 1 - Benchmark your Windows Device:

#### Quick Test 1: Benchmark to evaluate device memory bandwidth over DirectX 12.
```sh
$ python.exe -m autort.benchmark.memtest
  ...
  [1000/1000] AutoRT Device Memory Bandwidth: (Actual ~= 468.12 GB/s) (Theoretical ~= 561.75 GB/s)
```

#### Quick Test 2: Benchmark to evaluate device FP32 performance over DirectX 12.
```sh
$ python.exe -m autort.benchmark.fp32test
  ...
  [5000/5000] AutoRT FP32 TFLOPS: (Actual ~= 9.84 TFLOPS) (Theoretical ~= 10.93 TFLOPS)

$ python.exe -m autort.benchmark.mmtest
  ...
```

#### Quick Test 3: Benchmark to evaluate device copy time.
```sh
$ python.exe -m autort.benchmark.copytest
  ...
```

#### Quick Test 4: Benchmark to evaluate device launch time.
```sh
$ python.exe -m autort.benchmark.launchtest
  ...
```

## - Playground 2 - Running Pytorch2 over DirectX:

#### Quick Test 1: Create "custom operator" of your own in Pytorch 2.

- **Style-1: "AutoRT API Style"** Custom Operator Generation:
```py
>> import torch, autort
>> data = torch.arange(0, 10, dtype=torch.float32, device=autort.device())

>> f = autort.export(ir="sigmoid_f32[N] = 1 - 1 / (1 + data[N].call(strs.exp))", inputs=["data=float32[N:4096000]"], config="tune:5")
>> print(f(data))
tensor([0.5000, 0.7311, 0.8808, 0.9526, 0.9820, 0.9933, 0.9975, 0.9991, 0.9997, 0.9999])
>> print(autort.ops.sigmoid_f32(data))
tensor([0.5000, 0.7311, 0.8808, 0.9526, 0.9820, 0.9933, 0.9975, 0.9991, 0.9997, 0.9999])
```

- **Style-2: "Command Line Style"** Custom Operator Generation:
```sh
# Fist, create a custom sigmoid activation operator with 5 tuning steps:
$ autort --ir "sigmoid_f32[N] = 1 - 1 / (1 + data[N].call(strs.exp))" -i data=float32[N:4096000] -c "tune:5"

# Then, use it in Pytorch 2 session:
$ python.exe
>> import torch, autort
>>
>> data = torch.arange(0, 10, dtype=torch.float32, device=autort.device())
>> output = autort.ops.sigmoid_f32(data)
>> print(output)
tensor([0.5000, 0.7311, 0.8808, 0.9526, 0.9820, 0.9933, 0.9975, 0.9991, 0.9997,
        0.9999])
>> output = torch.nn.functional.sigmoid(data)
>> print(output)
tensor([0.5000, 0.7311, 0.8808, 0.9526, 0.9820, 0.9933, 0.9975, 0.9991, 0.9997,
        0.9999])
```


#### Quick Test 2: Demo of Sorting/MNIST/LLama over Pytorch2:

```sh
$ python.exe -m autort.examples.01_sort_even_first

Input : tensor([101, 102, 208,  99,   1, 127,  62,   8, 336, 336], dtype=torch.int32)
  (is_even) tensor([False,  True,  True, False, False, False,  True,  True,  True,  True])

Output: tensor([102, 208,  62,   8, 336, 336, 101,  99,   1, 127], dtype=torch.int32)
  (is_even) tensor([ True,  True,  True,  True,  True,  True, False, False, False, False])
```

```sh
$ python.exe -m autort.examples.02_mnist
  ...
  step = 800, loss = 0.5159, accuracy = 87.50 %
  step = 900, loss = 0.5511, accuracy = 84.38 %
  step = 1000, loss = 0.2616, accuracy = 93.75 %
  ...
```

```sh
$ python.exe -m autort.examples.03_llama_tiny

What is that?"
"That is the sun," her mom said. "It gives us heat."
The little girl was amazed. She had never seen the heat before.
"Can we go outside and feel the sun?" she asked.
"Yes," her mother said.
...
```

```sh
$ python.exe -m autort.examples.05_llama2_7b_int4

How large is Atlantic Ocean?

The Atlantic Ocean is the second largest ocean on Earth, covering approximately 20% of the Earth's surface. ...
```

```sh
$ python.exe -m autort.examples.06_diffuser_no_opt

...
Image converted from `./samurai_nn.png` to `samurai_nn_diffused.png`..
```

If you like it, welcome to report issues or donate stars which can encourage AutoRT to support more backends, more OS-type and more documentations. See More Information about Microsoft [Contributing](CONTRIBUTING.md) and [Trademarks](TRADEMARKS.md).
