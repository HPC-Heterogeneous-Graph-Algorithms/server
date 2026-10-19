# Shared NVIDIA GPU Server — User Guide

This server has **one shared NVIDIA GPU**. Approved users can monitor it with `nvidia-smi` and change supported GPU clock, memory-clock, and power settings using **`gpu-smi` without typing sudo**.

> [!IMPORTANT]
> Changes to GPU settings affect **all users and workloads** on this GPU. Coordinate tuning experiments and restore previous settings when appropriate. You are authorized to use the tool, not to disrupt other users or access files that aren't yours.

## 1. Activate your access

Ask an administrator to add your Linux account to the `gpu_tuners` group. **Log out and reconnect SSH** after the administrator adds you.

Verify your current session:

```bash
id -nG
gpu-smi -L
```

`id -nG` should include `gpu_tuners`. `gpu-smi` should not request a sudo password.

## 2. Monitor GPU usage (no special permissions normally needed)

```bash
nvidia-smi                         # Current utilization and processes
nvidia-smi -L                      # GPU inventory
nvidia-smi -q                      # Detailed GPU information
nvidia-smi -q -d POWER             # Power limits and usage
nvidia-smi -q -d CLOCK             # Clock information
nvidia-smi -q -d SUPPORTED_CLOCKS  # Supported clock combinations
nvidia-smi --help                  # Available options
```

## 3. Change GPU clocks and power settings

Use `gpu-smi` for privileged operations. **The numbers below are examples, not recommended values**; check what the installed GPU and driver support.

```bash
# Lock graphics clocks to a supported range (MHz)
gpu-smi -i 0 -lgc 1200,1500

# Lock memory clock to a supported frequency (MHz), if available
gpu-smi -i 0 -lmc 5000,5000

# Set a supported power limit (watts)
gpu-smi -i 0 -pl 250

# Reset graphics clock locks
gpu-smi -i 0 -rgc

# Reset memory clock locks
gpu-smi -i 0 -rmc
```

The wrapper forwards arguments to `nvidia-smi` with elevated privileges. Other `nvidia-smi` options may therefore be available, but commands such as GPU resets, file outputs, or mode changes can be disruptive or security-sensitive. Do not use them without authorization and coordination.

## 4. Shared-server etiquette

- Confirm no one else is running a sensitive training or benchmarking job before tuning.
- Record the initial clocks and power limits before changes.
- Reset or restore settings after your experiment, where appropriate.
- Use only clock and power values supported by the installed GPU and driver.
- Do **not** use privileged file-output or administrative functionality to access other users' files.

## 5. Troubleshooting

**`gpu-smi: command not found`**

```bash
/usr/local/bin/gpu-smi -L
```

If that works, check your `PATH` or ask your administrator.

**`sudo: a password is required`**

```bash
id -nG
```

If `gpu_tuners` is absent, log out and reconnect SSH. If it is still absent, contact the administrator.

**Unsupported operation or invalid setting**

```bash
nvidia-smi -q -d POWER
nvidia-smi -q -d SUPPORTED_CLOCKS
nvidia-smi --help
```

Your GPU architecture or driver may not support the operation or the requested value.

## 6. Optional CPU profiling (admin-enabled separately)

Check the global profiling restriction:

```bash
cat /proc/sys/kernel/perf_event_paranoid
```

If an administrator has configured `2`, user-space profiling may be permitted:

```bash
perf stat -e cycles,instructions -- sleep 1
```

This `perf` configuration is independent of GPU access and the `gpu_tuners` group.

---

For account changes and sudoers configuration, see [README_ADMIN.md](README_ADMIN.md).
