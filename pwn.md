## **经典堆利用技术**

### ​**(1) Unlink Attack**

- ​**目标**：通过伪造 `fd` 和 `bk` 触发 `unlink` 宏，实现任意地址写入。
- ​**条件**：
    - 需溢出修改一个空闲 chunk 的 `fd` 和 `bk`。
    - 触发 `unlink`（如合并空闲块时）。
- ​**利用代码**：
    
    c
    
    复制
    
    ```c
    // 伪造 fd 和 bk 指向目标地址 - 0x18 和 -0x10
    chunk->fd = target - 0x18;
    chunk->bk = target - 0x10;
    // 触发 unlink 后：*(target) = target + 0x8
    ```
    

### ​**(2) Fastbin Attack**

- ​**目标**：篡改 `fastbin` 的 `fd` 指向恶意地址，分配后控制该地址。
- ​**步骤**：
    1. 溢出修改 `fastbin` chunk 的 `fd` 指向伪造 chunk。
    2. 两次 `malloc` 分配目标地址。
- ​**限制**：
    - 伪造地址需满足 `size` 校验（如 `0x7f`）。
    - 需绕过 `malloc` 的 `double-free` 检测。

### ​**(3) Unsortedbin Attack**

- ​**目标**：通过修改 `unsortedbin` 的 `bk` 实现任意地址写入一个大数。
- ​**利用**：
    - 覆盖 `bk` 为 `target_addr - 0x10`，触发后 `*(target_addr) = main_arena`。