# slvs_py — FreeBSD fork

This is a fork of [realthunder/slvs_py](https://github.com/realthunder/slvs_py)
with patches applied to make it build successfully on FreeBSD with CMake 3.31+.

The SolveSpace submodule (`py_slvs`) points to
[NeoTorino/solvespace](https://github.com/NeoTorino/solvespace) on the
`freebsd-cmake-fix` branch, which contains the CMake compatibility patch.

---

## What is this?

`slvs_py` is a Python wrapper around the [SolveSpace](https://solvespace.com)
geometric constraint solver. It is required by the
[CAD Sketcher](https://github.com/hlorus/CAD_Sketcher) addon for Blender.

---

## What changed from upstream

The upstream `py_slvs/CMakeLists.txt` (inside the solvespace submodule) declares:

```cmake
cmake_minimum_required(VERSION 3.1.0 FATAL_ERROR)
cmake_policy(VERSION 3.1.0)
```

CMake 3.31 removed backwards compatibility for version declarations below 3.5,
causing a hard build failure on any system running a modern CMake. Both lines
have been updated to `VERSION 3.5` in the
[NeoTorino/solvespace](https://github.com/NeoTorino/solvespace) fork on the
`freebsd-cmake-fix` branch.

The `.gitmodules` file in this repo has been updated to point to that fork and
branch so the patch is automatically applied when you initialize the submodules.

---

## Building on FreeBSD

### System requirements

- FreeBSD 14+
- Python 3.11
- CMake 3.5 or later (tested with 3.31.10)
- Clang (comes with FreeBSD base)

### Install build dependencies

```sh
pkg install swig ninja
```

Both packages are managed by `pkg` and will be updated with `pkg upgrade`.

### Clone the repo and initialize submodules

```sh
git clone https://github.com/NeoTorino/slvs_py.git
cd slvs_py/slvs_py
git submodule update --init --recursive
```

The submodule will check out the `freebsd-cmake-fix` branch from
[NeoTorino/solvespace](https://github.com/NeoTorino/solvespace), which already
contains the CMake patch. No manual patching is needed.

### Build and install

```sh
pip install .
```

### Create the import shim

CAD Sketcher does `import slvs` but the package installs as `py_slvs`, with the
actual module at `py_slvs.slvs`. Create a shim so the import resolves correctly:

```sh
echo "from py_slvs.slvs import *" > ~/.local/lib/python3.11/site-packages/slvs.py
```

### Verify

```sh
python3.11 -c "import slvs; print(slvs)"
```

You should see something like:

```
<module 'py_slvs.slvs' from '/home/USERNAME/.local/lib/python3.11/site-packages/py_slvs/slvs.py'>
```

---

## Using with CAD Sketcher in Blender

Blender 5.0 on FreeBSD uses the system Python (`/usr/local/bin/python3.11`),
the same Python that pip installs into, so no further configuration is needed.
After installing and creating the shim, restart Blender and CAD Sketcher will
detect the solver automatically.

Do not use the in-addon pip installer — it will fail on FreeBSD due to the same
CMake version issue this fork fixes. Do not attempt to install via the addon's
"install from file" option either, as the addon's post-install logic has a bug
that throws a NameError regardless of whether the install succeeded.

---

## Troubleshooting

### ModuleNotFoundError: No module named 'slvs'

The shim file is missing or in the wrong location. Find your user site-packages
path and recreate it:

```sh
echo "from py_slvs.slvs import *" > $(python3.11 -c "import site; print(site.getusersitepackages())")/slvs.py
```

### CMake errors during build

Verify the submodule was initialized correctly and the patch is in place:

```sh
grep -n 'cmake_minimum_required\|cmake_policy' py_slvs/CMakeLists.txt
```

Both lines should show `VERSION 3.5`.

### SWIG not found during build

```sh
pkg install swig
```

### Checking shared library dependencies of the built extension

```sh
ldd ~/.local/lib/python3.11/site-packages/py_slvs/_slvs.so
```

All entries should show a resolved path. All required libraries (`libc++`,
`libcxxrt`, `libm`, `libgcc_s`, `libc`) are present in base FreeBSD and require
no extra packages.

---

## Tested on

- FreeBSD 14.3-RELEASE-p10 amd64
- Clang 19.1.7
- CMake 3.31.10
- Python 3.11.15
- Blender 5.0

---

## Upstream

This fork tracks [realthunder/slvs_py](https://github.com/realthunder/slvs_py).
The solvespace submodule patches track
[realthunder/solvespace](https://github.com/realthunder/solvespace).
For issues unrelated to FreeBSD compatibility, please report them upstream.
