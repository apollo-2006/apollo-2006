```
  ●  research      learning ml, writing it down
  │╲
  │ ●  passions    lifting · mma · gaming · reading
  │  ╲
  │   ●  projects  16 systems, built raw
  │  ╱
  │ ●  work        u of a · grad 2028
  │╱
  ●  connect       say hello
```

# abir deol

Second year computing science at the University of Alberta. I write low level systems
from scratch to find out what is underneath the abstraction: an allocator instead of
`malloc`, Raft instead of a database that already handles consensus, a rasterizer
instead of a graphics API.

None of it is better than the real thing. That was never the point. The point is that
afterwards the real thing stops being magic, so when it behaves strangely you have
somewhere to start.

**[abirdeol.tech](https://abirdeol.tech)** &nbsp;·&nbsp; [resume](https://abirdeol.tech/abir-deol-resume.pdf) &nbsp;·&nbsp; [live demo](https://apollo-2006.github.io/cpu_rasterizer/) &nbsp;·&nbsp; [linkedin](https://linkedin.com/in/abirdeol)

---

### distributed systems and storage

- **[nexus_db](https://github.com/apollo-2006/nexus_db)** &nbsp; embedded LSM key-value store: skip list memtable, write ahead log, immutable SSTables, tombstone compaction &nbsp;`C++17` `FastAPI` `React`
- **[nexus_cluster](https://github.com/apollo-2006/nexus_cluster)** &nbsp; Raft from scratch: leader election, log replication, term management, custom TCP transport &nbsp;`Python`
- **[nexus_editor](https://github.com/apollo-2006/nexus_editor)** &nbsp; real time collaborative editor, fractional indexing CRDT, goroutine per client &nbsp;`Go` `React` `TS`

### close to the metal

- **[nano_match](https://github.com/apollo-2006/nano_match)** &nbsp; limit order book with price time priority and no heap allocation on the matching path &nbsp;`C++`
- **[custom_mem_alloc](https://github.com/apollo-2006/custom_mem_alloc)** &nbsp; thread safe allocator over one `mmap` region, intrusive free list, block recycling &nbsp;`C` `pthreads`
- **[neon_vm](https://github.com/apollo-2006/neon_vm)** &nbsp; stack based virtual machine, bytecode chunking, its own dispatch loop &nbsp;`C`

### graphics

- **[cpu_rasterizer](https://github.com/apollo-2006/cpu_rasterizer)** &nbsp; full 3D pipeline in plain JavaScript, no graphics API. barycentric fill, `1/w` z-buffer. **[runs in your browser](https://apollo-2006.github.io/cpu_rasterizer/)** &nbsp;`JS` `React`
- **[photon_tracer](https://github.com/apollo-2006/photon_tracer)** &nbsp; raytracer with no libraries: vector math, ray generation, camera, PPM output &nbsp;`C++`
- **[rasterizer_engine](https://github.com/apollo-2006/rasterizer_engine)** &nbsp; software rasterizer, perspective matrices and triangle fill written out longhand &nbsp;`C++17` `SDL2`

### tools I actually use

- **[terminal_dashboard](https://github.com/apollo-2006/terminal_dashboard)** &nbsp; system monitor with per core CPU, memory, network and GPU telemetry &nbsp;`Python` `Rich`
- **[points-sys](https://github.com/apollo-2006/points-sys)** &nbsp; two player points economy, one HTML file, no build step, live synced &nbsp;`Firestore`
- **[cal-cli](https://github.com/apollo-2006/cal-cli)** &nbsp; macro tracking from the command line, because logging a meal should be one command &nbsp;`Python`

---

### things I got wrong

Four of the projects above had a bug that cost me real hours. I wrote each one up
afterwards: the symptom, how I chased it, what was actually wrong, and what I changed
about how I write that kind of code.

- **[The block that was too small to free](https://abirdeol.tech/research/allocator)** &nbsp; an intrusive free list corrupting the block next door
- **[The delete that did not delete](https://abirdeol.tech/research/tombstones)** &nbsp; a flush optimization that resurrected deleted keys
- **[The order that was in two places](https://abirdeol.tech/research/order-pool)** &nbsp; a use after free that never crashed
- **[A cluster that could not keep a leader](https://abirdeol.tech/research/election-timeouts)** &nbsp; when the runtime pauses longer than your failure detector waits

### currently

Working through machine learning from classical computer vision upward, and
[writing down what I learn](https://abirdeol.tech/research) rather than collecting
tutorials. Two-time national MMA champion, 2020 to 2024. Looking for software
engineering internships.
