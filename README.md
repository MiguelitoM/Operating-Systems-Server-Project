# IST‑KVS – Operating Systems Project
IST‑KVS (Instituto Superior Técnico – Key Value Store) is a small‑scale data store developed as part of the Operating Systems course. It was implemented in two incremental parts that gradually introduce POSIX file operations, multi‑threading, inter‑process communication, and signal handling.

## 🏦 Part 1 – Key–Value Store
In the first part the focus is on offline processing of job files that contain simple textual commands (`WRITE`, `READ`, `DELETE`, `SHOW`, `WAIT`, `BACKUP`) over an in‑memory hash table. Highlights:

* **Execution** – the program scans a directory, picks every `.job` file and streams the commands inside, writing the results to a matching `.out` file.
* **Non‑blocking backups** – each `BACKUP` spawns a child process so the main workflow keeps running while the snapshot is written to `<jobname>-<n>.bck`, with a configurable upper bound on concurrent backups.
* **Parallelism** – different job files are processed in parallel by a pool of worker threads; fine‑grained locks protect the shared hash table.

## 🔗 Part 2 – Server & Client Subscriptions
The second milestone turns IST‑KVS into a long‑running server that external programs can monitor in real time. Main ideas:

* **Named pipes interface** – clients connect through a registration FIFO, then use dedicated request / response / notification FIFOs to interact with the server.
* **Live subscriptions** – clients may `SUBSCRIBE` or `UNSUBSCRIBE` to keys and get asynchronous updates whenever those keys change.
* **Scalable architecture** – a host thread accepts sessions while a fixed set of worker threads serve requests concurrently. The original job‑processing threads from Part 1 keep running side‑by‑side.
* **Graceful mass disconnect** – sending `SIGUSR1` to the server closes every open FIFO, clears all subscriptions and leaves the server ready for fresh connections.

## ⚙️ Compiling
```bash
make        # builds everything
make clean  # removes generated files
```

## 🕹 Running
### Batch mode (Part 1)
```bash
./kvs <jobs_dir> <max_backups> <max_threads>
```

### Server mode (Part 2)
```bash
# Start the server
./kvs <jobs_dir> <max_threads> <max_backups> <registration_fifo>

# Launch a client
./client <client_id> <registration_fifo>   # use commands on stdin
```

## 📖 Learning outcomes
* Low‑level file I/O with POSIX descriptors
* Thread synchronisation using mutexes & semaphores
* Inter‑process communication with FIFOs and UNIX signals
* Designing producer–consumer pipelines
