# jupyter-kernels

Dependency-free Python 3 utility for Linux. Installed at `~/.local/bin/jupyter-kernels`.

```bash
jupyter-kernels                       # running kernels, host PIDs, ROS evidence
jupyter-kernels --details             # commands, cwd, evidence per descendant
jupyter-kernels -i                    # choose PID and confirm TERM
jupyter-kernels --kill 12345          # confirm then terminate kernel
jupyter-kernels --kill 12345 --tree   # include current descendants
jupyter-kernels --kill 12345 -s INT   # interrupt (kernel may keep running)
jupyter-kernels --kill 12345 -s KILL  # force termination
jupyter-kernels --kill 12345 --tree -y
jupyter-kernels --json
```

Lists running processes rather than installed kernel specifications or stale
connection files. Detects Python ipykernel launches and other kernels with a
`-f` / `--connection-file` argument pointing to a kernel JSON file. Unusual
launch commands may not be detected. It does not infer whether a frontend is
still attached. Age is process age, not notebook idle time.

ROS evidence includes the kernel and all visible descendants:

- `loaded`: ROS client/middleware libraries appear in process memory maps.
- `command`: a recognized ROS executable appears in the argument list.
- `env-only`: ROS setup variables were inherited; ROS may never have been used.
- `none-seen`: no evidence found, which does not rule out ROS activity.
- `unknown`: process maps or environment could not be read.

Even loaded libraries do not prove an active ROS node. Evidence is heuristic;
unrelated ROS nodes and sibling processes are not associated with a kernel.

Run from a host terminal to see host and Docker processes. Inside a container or
restricted sandbox only processes visible in its PID namespace can be listed.
Docker cgroups supply a short container ID when available; a Docker root marker
supplies a generic container label otherwise. `host/?` means no marker was found.
Permissions may restrict processes, ROS evidence, or signals. Running with sudo
can reveal other users' processes when needed.

Signals target the host PID, never the entire Docker container or process group.
Only detected kernels can be selected. Descendants require `--tree`; its snapshot
cannot capture children created later or children already reparented away.
Signals check process start time and use pidfds when supported to prevent PID
reuse mistakes. Supervisors such as Jupyter may restart a terminated kernel.
SIGKILL skips process cleanup. No kernels are signaled during listing.
