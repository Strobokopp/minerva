# Minerva

## A 32-bit RISC-V soft processor

Minerva is a CPU core that currently implements the [RISC-V][1] RV32IM instruction set. Its microarchitecture is described in plain Python code using the [nMigen][2] toolbox.

### Quick start

Minerva currently requires Python 3.6+ and [nMigen][2] on its `master` branch.

    python setup.py install
    python build.py > minerva.v

To use Minerva, you need to wire the following ports to `minerva_cpu`:

* `clk`
* `rst`
* `ibus__*`
* `dbus__*`
* `external_interrupt`

### Features

The microarchitecture of Minerva is largely inspired by the [LatticeMico32][3] processor.

Minerva is pipelined on 6 stages:

1. **Address**
   The address of the next instruction is calculated and sent to the instruction cache.
2. **Fetch**
   The instruction is read from memory.
3. **Decode**
   The instruction is decoded, and operands are either fetched from the register file or bypassed from the pipeline. Branches are predicted by the static branch predictor.
4. **Execute**
   Simple instructions such as arithmetic and logical operations are completed at this stage.
5. **Memory**
   More complicated instructions such as loads, stores and shifts require a second execution stage.
6. **Writeback**
   Results produced by the instructions are written back to the register file.

![Pipeline Diagram Image](https://docs.google.com/drawings/d/e/2PACX-1vTMkQc8ZJoiJ2AOeFGMkK0QTNx1hSG5wDrG5seLdJ3i61E4ag7wH7VFey44qhvuXotvOKxOw-mFS-VE/pub?w=850&h=761)

The two L1 caches are write-through, meaning that writes are done to both the cache and the underlying memory hierarchy.

Miss penalties are reduced by restarting execution as soon as the missing word is available, without waiting for the complete line refill. Furthermore, cache refills always request the missing word first. By combining these two policies, Minerva is able to resume its execution after a cache miss in only *two clock cycles* in the best case, regardless of line size.

The L1 data cache is coupled to a write buffer. Store transactions are in this case done to the write buffer instead of the data bus. This enables stores to proceed in one clock cycle if the buffer isn't full, without having to wait for the bus transaction to complete. Store transactions are then completed in the background as the write buffer gets emptied to the data bus.

Minerva is able to operate at 100MHz on a Xilinx XC7A35-2 FPGA, with both caches enabled.

### Configuration

The following parameters can be used to configure the Minerva core.

| Parameter         | Default value  | Description                                        |
| ----------------- | -------------- | -------------------------------------------------- |
| `reset_address`   | `0x00000000`   | Reset vector address                               |
| `with_icache`     | `True`         | Enable the instruction cache                       |
| `icache_nb_ways`  | `1`            | Number of ways in the instruction cache            |
| `icache_nb_lines` | `256`          | Number of lines in the instruction cache           |
| `icache_nb_words` | `8`            | Number of words in a line of the instruction cache |
| `icache_base`     | `0x00000000`   | Base of the instruction cache address space        |
| `icache_limit`    | `0x80000000`   | Limit of the instruction cache address space       |
| `with_dcache`     | `True`         | Enable the data cache                              |
| `dcache_nb_ways`  | `1`            | Number of ways in the data cache                   |
| `dcache_nb_lines` | `256`          | Number of lines in the data cache                  |
| `dcache_nb_words` | `8`            | Number of words in a line of the data cache        |
| `dcache_base`     | `0x00000000`   | Base of the data cache address space               |
| `dcache_limit`    | `0x80000000`   | Limit of the data cache address space              |
| `as_instance`     | `False`        | Add a default clock domain                         |
| `with_muldiv`     | `False`        | Enable RV32M support                               |
| `with_debug`      | `False`        | Enable the Debug Module                            |
| `with_trigger`    | `False`        | Enable the Trigger Module                          |

### Possible improvements

In no particular order:

* RV64I
* Floating Point Unit
* Stateful branch prediction
* ...

If you are interested in sponsoring new features or improvements, get in touch at contact [at] lambdaconcept.com .

### License

Minerva is released under the permissive two-clause BSD license.
See LICENSE file for full copyright and license information.

[1]: https://riscv.org/specifications/
[2]: https://github.com/m-labs/nmigen/
[3]: https://github.com/m-labs/lm32/
