# linux_kernal_project-
# Linux Kernel Drivers

Loadable Linux kernel modules written in C, with a userspace test for each.

| Module | What it shows |
|---|---|
| `hello/` | Minimal module: `module_init` / `module_exit`, `pr_info`, module metadata |
| `ringdev/` | Character device `/dev/ringdev` backed by a kernel ring buffer |

## ringdev

A `misc` character driver (`misc_register`, dynamic minor).

- Mutex-protected ring buffer; size set with the `bufsize` module parameter (default 4096 bytes)
- `read` / `write` that block on wait queues, or return `-EAGAIN` with `O_NONBLOCK`
- `poll()`: `POLLIN` when data is available, `POLLOUT` when there is space
- `ioctl`: `RINGDEV_CLEAR` (empty the buffer), `RINGDEV_GET_COUNT` (bytes stored); other commands return `-ENOTTY`
- Clean unload: `misc_deregister` and `kfree`

## Build and run

Needs a real Linux kernel with headers (Ubuntu, a VM or a Raspberry Pi). WSL does not work.

```bash
sudo apt install build-essential linux-headers-$(uname -r)

cd hello && make
sudo insmod hello.ko && sudo dmesg | tail -n 3 && sudo rmmod hello

cd ../ringdev && make
sudo insmod ringdev.ko bufsize=16
sudo chmod 666 /dev/ringdev
gcc -Wall -o test_ringdev test_ringdev.c && ./test_ringdev
sudo rmmod ringdev && sudo dmesg | tail -n 3
```

Expected: every line prints `PASS` and the last line is `RESULT: ALL PASSED`.

## Test status

- Built against Linux 6.12 and loaded into a User-Mode Linux kernel.
- 18 checks passed: `insmod`, write, read, count, empty and full `EAGAIN`, `poll`, wraparound, `CLEAR`, `ENOTTY`, `rmmod`.
- Not yet run on a physical machine or a distribution kernel.

## Author

Manuhasa M · github.com/manuhasampostbox-collab
