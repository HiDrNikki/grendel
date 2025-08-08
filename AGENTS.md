# ./grendel/AGENTS.md

The project directory has the following structure:

grendel/
├─ grendel/
│  ├─ benchmarks/
│  │  ├─ __init__.py
│  │  ├─ perfHarness.py
│  │  └─ syntheticKernels.py
│  ├─ docs/
│  │  ├─ api.md
│  │  ├─ architecture.md
│  │  ├─ memoryModel.md
│  │  ├─ roadmap.md
│  │  └─ shim.md
│  ├─ examples/
│  │  ├─ cupyDemo.py
│  │  └─ minimalClient.py
│  ├─ scripts/
│  │  ├─ buildShim.ps1
│  │  ├─ buildShim.sh
│  │  ├─ dev.ps1
│  │  └─ dev.sh
│  └─ src/
│     └─ grendel/
│        ├─ api/
│        │  ├─ __init__.py
│        │  ├─ models.py
│        │  ├─ restServer.py
│        │  └─ rpcServer.py
│        ├─ config/
│        │  ├─ __init__.py
│        │  ├─ defaults.toml
│        │  ├─ loader.py
│        │  └─ schema.json
│        ├─ core/
│        │  ├─ policy/
│        │  │  ├─ __init__.py
│        │  │  ├─ kvStore.py
│        │  │  └─ placementPolicy.py
│        │  ├─ __init__.py
│        │  ├─ daemon.py
│        │  ├─ deviceDiscovery.py
│        │  ├─ jobManager.py
│        │  ├─ scheduler.py
│        │  └─ topology.py
│        ├─ cython/
│        │  ├─ gpu/
│        │  │  ├─ fastMemcpy.pxd
│        │  │  └─ fastMemcpy.pyx
│        │  ├─ memory/
│        │  │  ├─ mappedHostPinned.pxd
│        │  │  ├─ mappedHostPinned.pyx
│        │  │  ├─ vramCache.pxd
│        │  │  └─ vramCache.pyx
│        │  ├─ __init__.py
│        │  └─ setupCython.py
│        ├─ gpu/
│        │  ├─ __init__.py
│        │  ├─ events.py
│        │  ├─ streams.py
│        │  └─ worker.py
│        ├─ ipc/
│        │  ├─ proto/
│        │  │  └─ grendel.proto
│        │  ├─ __init__.py
│        │  ├─ client.py
│        │  ├─ messages.py
│        │  └─ server.py
│        ├─ memory/
│        │  ├─ __init__.py
│        │  ├─ dedupe.py
│        │  ├─ grendelRam.py
│        │  ├─ nvmeSpill.py
│        │  ├─ pager.py
│        │  ├─ prefetcher.py
│        │  ├─ slice.py
│        │  └─ vramCache.py
│        ├─ observability/
│        │  ├─ __init__.py
│        │  ├─ logging.py
│        │  ├─ metrics.py
│        │  └─ tracing.py
│        ├─ shim/
│        │  ├─ cudaShim.c
│        │  ├─ cudaShim.h
│        │  ├─ meson.build
│        │  └─ README.md
│        ├─ tools/
│        │  ├─ __init__.py
│        │  └─ grendelCtl.py
│        ├─ __init__.py
│        ├─ __main__.py
│        ├─ cli.py
│        └─ version.py
├─ src/
│  └─ grendel/
│     ├─ __init__.py
│     ├─ __main__.py
│     └─ cli.py
├─ AGENTS.md
├─ CHANGELOG.md
├─ LICENSE
├─ pyproject.toml
├─ README.md
└─ setup.py