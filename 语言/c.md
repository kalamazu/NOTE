#mmap 
mmap(addr,size,permission,flag,-1,offset)
permission=7,具有读写执行权限

#j_memcpy
memcpy(addr_destination，addr_source，size_t_number)

#read
ssize_t read(int fd, void * buf, size_t count);
**参数**:

- `rdi`: 文件描述符 `fd` (0 = stdin, 1 = stdout, 2 = stderr)
- `rsi`: 目标缓冲区 `buf`
- `rdx`: 读取的字节数 `count`

#write
ssize_t write(int fd, const void * buf, size_t count);
**参数**:

- `rdi`: 文件描述符 `fd` (1 = stdout)
- `rsi`: 缓冲区地址 `buf`
- `rdx`: 要写入的字节数 `count`

#open
int open(const char * pathname, int flags, mode_t mode);

**参数**:

- `rdi`: 文件路径 `pathname`
- `rsi`: 打开方式 `flags`（`O_RDONLY=0`, `O_WRONLY=1`, `O_RDWR=2`）
- `rdx`: 权限 `mode`（创建文件时需要，如 `0644`）

#close
int close(int fd);

#mmap 
void * mmap(void * addr, size_t length, int prot, int flags, int fd, off_t offset);

**参数**:

- `rdi`: 目标地址 `addr`（`NULL` 让系统选择）
- `rsi`: 映射大小 `length`
- `rdx`: 保护标志 `prot`（`PROT_READ=1, PROT_WRITE=2, PROT_EXEC=4`）
- `r10`: 标志 `flags`（`MAP_PRIVATE=2, MAP_ANONYMOUS=32`）
- `r8`: 文件描述符 `fd`（通常 `-1`）
- `r9`: 偏移量 `offset`


#execve
int execve(const char * pathname, char * const argv[], char * const envp[]);

**参数**:

- `rdi`: 可执行文件路径（如 `/bin/sh`）
- `rsi`: 参数数组（通常 `NULL`）
- `rdx`: 环境变量数组（通常 `NULL`）

#fork
创建子进程 pid_t fork(void);
