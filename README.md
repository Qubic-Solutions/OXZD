# OXZD Multi Miner (CUDA + CPU)

OXZD is a high performance CPU + GPU miner made in Rust. The main goal of this miner is to have a simple scalable framework for all sorts of algorithms on CPU and GPU.

[![](https://img.shields.io/discord/1179806757204267090?color=5865F2&logo=Discord&style=flat-square)](https://discord.poolhub.io)

Overview
- [Algo Config](#algo-config)
- [Idle Algos](#idle-algorithms--command-algos)
- [CPU Config](#cpu)
- [GPU Config](#gpu)
- [Commands](#commands)
- [Example Config](#how-to-run-oxzd-example)

## Configuration

If no configuration is provided the miner will automatically generate a new file in the current folder containing a configuration for all algorithms currently available in OXZD.

<pre>
[<span style="color:blue">2025/06/11 13:40:49</span>  <span style="color:magenta">OXZD</span>  0.5.4] <span style="color:yellow">WARN</span> Configuration file oxzd_config.json does not exist, generating default configuration
</pre>

```json
{
  "selected": [
  ],
  "algo_list": [
    {
      "id": "xelis-v2-template",
      "algo": "xelis_v2",
      "pool": "xelis-v2.poolhub.me:5077",
      "worker_name": "7950X_1",
      "address": "3QpJZJY1J",
      "config": {
        "type": "cpu",
        "threads": 16
      },
      "password": null,
      "idle_algos": null
    },
    {
      "id": "tip5-template",
      "algo": "neptune",
      "pool": "tip5.poolhub.me:5077",
      "worker_name": "7950X_1",
      "address": "3QpJZJY1J",
      "config": {
        "type": "gpu",
        "option": "all"
      },
      "password": null,
      "idle_algos": null
    },
    {
      "id": "randomx-template",
      "algo": "random_x",
      "pool": "randomx.poolhub.me:5077",
      "worker_name": "7950X_1",
      "address": "3QpJZJY1J",
      "config": {
        "type": "cpu",
        "threads": 16
      },
      "password": null,
      "idle_algos": null
    },
    {
      "id": "tari-sha3x-template",
      "algo": "tari_sha3x",
      "pool": "tari-sha3x.poolhub.me:5077",
      "worker_name": "7950X_1",
      "address": "3QpJZJY1J",
      "config": {
        "type": "gpu",
        "option": "all"
      },
      "password": null,
      "idle_algos": null
    }
  ]
}
```

The config consists of 2 parts, the `selected` field selects the algo configurations in `algo_list` by `id` and runs them when the miner is launched with the default  [`./oxzd run` command](#oxzd-run--oxzd-bench).

### `algo_list`

The actual configurations are inside `algo_list` array, which can hold as many algo configs as needed. Configs are distinguished by the `id` field in the miner, this means that the same algorithm can be added multiple times, with different `id` and different values. In order to launch the configs wanted the `id` either need to be part of the `selected` array or [overridden in the launch command](#override-algo-selection).

### Algo Config

In general a config looks like this:

```json
{
    "id": "algo-id",
    "algo": "algo_name",
    "pool": "stratum+tcp://my-pool.io:PORT",
    "worker_name": "my-worker-name",
    "address": "my-address",
    "config": {},
    "password": null,
    "idle_algos": null
}
```

The `password` and `idle_algos` field are optional since only some algorithms support them, they can be omitted for better readability.

#### Idle Algorithms & Command Algos

Since some algorithms don't require the miner to be active 100% of the time, idle algos can be added to bridge that time. Any other algo config qualifies as idle algo, note that chaining idle algos is not yet supported. In addition to native algorithms, commands can also be used to mine the preferred coin even if it's not natively implemented in OXZD.

Example for command config
```json
{
    "id": "command-miner",
    "command": "./other-miner"
}
```

A command is treated like any other algo config in the list, selection is done via `id` and the command will be run as child process and closed if selected as idle algo and when the parent algo starts working again.

To run any algo config or command as idle algorithm, the `idle_algos` field needs to be changed.

```json
{
    "id": "main-algo",
    "algo": "algo_name",
    "pool": "stratum+tcp://my-pool.xyz:PORT",
    "worker_name": "my-worker-name",
    "address": "my-address",
    "config": {},
    "idle_algos": ["other-algo-or-command"]
}
```

The algo/command will now be run during idle phases of the config!

#### CPU

CPU & GPU [inherit the same structure shown above](#algo-config). They only differ in the `config` field, There are multiple ways to configure your algorithm for CPUs, most importantly the [thread selection](#thread-selection) and hugepage/largepage support for Linux & Windows.

```json
{
    "config": {
        "type": "cpu",
        "threads": 16,
        "hugepages": null
    }
}
```

- `type` is only used to distinguish between CPU and GPU, the only valid options are `"cpu"` and `"gpu"`

#### Thread Selection

The thread selection defines which threads should be selected to run the algorithm on. Currently there are 4 options

- `threads`: Selects the first N threads. Example: `"threads": 16`
- `last`: Selects the last N threads.     Example: `"last": 16`
- `select`: Selects the threads specified. Example: `"select": [0, 2, 4, 6, 8, 10, 12, 14]`
- `exclude`: Selects the threads that are **NOT** specified. Example: `"exclude": [1, 3, 4, 5, 7, 9, 11, 13, 15]`

#### Hugepage/Largepage Support

The `hugepages` option only affects algorithms that support its use. The option can be omitted.

Example:

`"hugepages": true`

#### GPU

The config field for GPU allows for the same customized selection as on CPU.

```json
{
    "config": {
        "type": "gpu",
        "option": "all"
    }
}
```

- `"option": "all"` Selects all GPUs available
- `gpus`: Selects the first N GPUs. Example: `"gpus": 2`
- `last`: Selects the last N GPUs.     Example: `"last": 2`
- `select`: Selects the GPUs specified. Example: `"select": [0, 1]`
- `exclude`: Selects the GPUs that are **NOT** specified. Example: `"exclude": [3, 4]`

## Commands

Using `./oxzd help` all available commands can be listed.

<pre>
<strong><u>Usage:</u> oxzd.exe</strong> &ltCOMMAND&gt

<strong><u>Commands:</u></strong>
  <strong>run</strong>          Run the miner with the specified configuration
  <strong>bench</strong>        Benchmark the miner locally with the specified configuration
  <strong>algo-list</strong>    List all available algorithms
  <strong>system-info</strong>  List system information
  <strong>help</strong>         Print this message or the help of the given subcommand(s)

<strong><u>Options:</u></strong>
  <strong>-h, --help</strong>   Print help
</pre>

### `oxzd run` & `oxzd bench`

The difference between run and bench is that bench does not connect to a pool and only runs locally to provide performance data.

`./oxzd run --help` (analogical `./oxzd bench --help`)
<pre>
<strong><u>Usage:</u> oxzd.exe run</strong> [OPTIONS]

<strong><u>Commands:</u></strong>
  <strong>-c, --config</strong> &ltCONFIG&gt   (Optional) Custom configuration file path, default: ./oxzd_config.json
  <strong>-a, --algos</strong> &ltALGOS&gt...  (Optional) Select algos to run by id
  <strong>-h, --help</strong>   Print help
</pre>

#### Override Config File Path

To use the preferred path for the config file the `-c/--config` option can be used.

Example:

`./oxzd run --config /my/config/path/config-name.json`

#### Override Algo Selection

For convenience algos can be selected through the run command by adding the `-a/--algos` option to the command. It's functionally the same as the [`selected` field](#configuration) in the config file.

Example:

`./oxzd run --algos algo-or-command-id1, algo-or-command-id2, algo-or-command-id3`

### `oxzd algo-list`

This command gives an overview on supported algos and features.

Example:

`./oxzd algo-list`

Output (0.5.4):
```
| Name                 | Type                 | Version              | Protocol             | Fee                  | Cluster Compatability     | Description
| xelis-v2             | CPU                  | 1.0.0                | Stratum              | 0.00%                | No                        | Xelis v2 hash algorithm
| tip5                 | GPU                  | 1.0.3                | Stratum              | 10.00%               | No                        | Neptune hash algorithm
| randomx              | CPU                  | 1.0.0                | Stratum              | 0.00%                | No                        | RandomX hash algorithm
| tari-sha3x           | GPU                  | 1.0.2                | Stratum              | 0.00%                | No                        | Tari SHA3X hash algorithm
```

### `oxzd system-info`

Shows general system information of the system as well as the binary optimization of the miner.

Example:

`./oxzd system-info`


Output (0.5.4):
```
OS:         Windows 10 (19045)
CPU:        AMD Ryzen 9 7950X 16-Core Processor
Memory:     30.92 GB / 63.15 GB
Arch:       x86_64
Binary Opt: znver4
CUDA:       yes
GPU0:       NVIDIA GeForce RTX 4090 23.99GiB
```

Note that the binary opt is relevant for CPU performance. The only exception is RandomX which can dynamically optimize its runtime based on CPU features supported. Multiple binaries for the miner are available to enable optimal performance for all CPU architectures.


## How to Run OXZD (Example)

For this example tip5 (Neptune) will be used, in general this will be similar for all other algos as well.

For more customized configs refer to [Configuration](#configuration)

`oxzd_config.json`
```json
{
  "selected": [],
  "algo_list": [
    {
        "id": "xelis-cpu",
        "algo": "xelis_v2",
        "pool": "stratum+ssl://de.vipor.net:5177",
        "worker_name": "7950X_1",
        "address": "xel:...",
        "config": {
          "type": "cpu",
          "exclude": [0, 1]
        }
    },
    {
      "id": "neptune-gpu",
      "algo": "neptune",
      "pool": "stratum+ssl://eu.poolhub.io:4444",
      "worker_name": "4090_1",
      "address": "nolgam10...",
      "config": {
        "type": "gpu",
        "option": "all"
      },
      "idle_algos": []
    },
    {
        "id": "tari-gpu",
        "algo": "tari_sha3x",
        "pool": "stratum+tcp://ca.luckypool.io:6118",
        "worker_name": "4090_1",
        "address": "12Am...",
        "password": "abcd",
        "config": {
          "type": "gpu",
          "option": "all"
        }
    },
    {
        "id": "rigel-xel",
        "command": "./rigel -a xelishashv2 -o stratum+ssl://de.vipor.net:5177 -u xel:... --no-tui"
    }
  ]
}
```

The config now contains 4 algorithms and 1 command. In order to select them they can be put into the `selected` array or [selected directly through the launch command](#override-algo-selection).

> Note: When running the same algorithm on CPU & GPU a shared connection to the pool is used, settings like pool, worker name, address and password are selected from the algo appearing first in the list

Now that the config is ready, the miner can be launched:

`./oxzd run --algos neptune-cpu, neptune-gpu`

Since neptune spends a long time doing nothing (and CPU cannot compete with GPU in TIP5) the config can be changed to improve profitability.

`oxzd_config.json`
```json
{
  "selected": [],
  "algo_list": [
    {
        "id": "xelis-cpu",
        "algo": "xelis_v2",
        "pool": "stratum+ssl://de.vipor.net:5177",
        "worker_name": "7950X_1",
        "address": "xel:...",
        "config": {
          "type": "cpu",
          "exclude": [0, 1]
        }
    },
    {
      "id": "neptune-gpu",
      "algo": "neptune",
      "pool": "stratum+ssl://eu.poolhub.io:4444",
      "worker_name": "4090_1",
      "address": "nolgam10...",
      "config": {
        "type": "gpu",
        "option": "all"
      },
      "idle_algos": ["rigel-xel"]
    },
    {
        "id": "tari-gpu",
        "algo": "tari_sha3x",
        "pool": "stratum+tcp://ca.luckypool.io:6118",
        "worker_name": "4090_1",
        "address": "12Am...",
        "password": "abcd",
        "config": {
          "type": "gpu",
          "option": "all"
        }
    },
    {
        "id": "rigel-xel",
        "command": "./rigel -a xelishashv2 -o stratum+ssl://de.vipor.net:5177 -u xel:... --no-tui"
    }
  ]
}
```

When `neptune-gpu` is launched now the idle algo (or command) will be launched in the background activating when neptune-gpu signals an idle phase and stopping when neptune-gpu receives new work.

`./oxzd run --algos neptune-gpu, xelis-cpu`