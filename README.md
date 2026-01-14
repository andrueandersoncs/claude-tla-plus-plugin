# TLA+ Plugin for Claude Code

A Claude Code plugin that enables generation of TLA+ specifications, PlusCal algorithms, and TLC model configurations for formal verification of distributed systems, concurrent algorithms, and state machines.

## Installation

### Local Development
```bash
claude --plugin-dir ./claude-tla-plus-plugin
```

### From Marketplace (once published)
```
/plugin install tla-plus@marketplace-name
```

## Features

### Skill: TLA+ Generator
Claude automatically uses the TLA+ generation skill when you ask about:
- Formal specification of systems
- Distributed systems modeling
- Concurrent algorithm verification
- State machine specification
- Safety and liveness properties
- Model checking configuration

### Slash Commands

| Command | Description |
|---------|-------------|
| `/tla-plus:spec <description>` | Generate a complete TLA+ specification |
| `/tla-plus:pluscal <description>` | Generate a PlusCal algorithm |
| `/tla-plus:invariant <description>` | Generate invariants for a spec |
| `/tla-plus:liveness <description>` | Generate liveness properties |
| `/tla-plus:config <description>` | Generate TLC configuration |
| `/tla-plus:review <file>` | Review an existing TLA+ spec |

## Usage Examples

### Generate a New Specification
```
/tla-plus:spec a simple key-value store with get and put operations
```

### Generate a Concurrent Algorithm
```
/tla-plus:pluscal Dijkstra's mutual exclusion algorithm for 3 processes
```

### Add Properties to Existing Spec
```
/tla-plus:invariant for my KeyValueStore.tla specification
/tla-plus:liveness ensure all requests eventually complete
```

### Generate Model Checker Config
```
/tla-plus:config for KeyValueStore.tla with 2 keys and 3 transactions
```

### Review a Specification
```
/tla-plus:review check my consensus protocol for issues
```

## Plugin Structure

```
claude-tla-plus-plugin/
├── .claude-plugin/
│   └── plugin.json           # Plugin manifest
├── commands/
│   ├── spec.md               # Generate specifications
│   ├── pluscal.md            # Generate PlusCal algorithms
│   ├── invariant.md          # Generate invariants
│   ├── liveness.md           # Generate liveness properties
│   ├── config.md             # Generate TLC configuration
│   └── review.md             # Review specifications
├── skills/
│   └── tla-plus-generator/
│       ├── SKILL.md          # Main skill definition
│       ├── syntax-reference.md    # Complete TLA+ syntax
│       ├── patterns-examples.md   # Example specifications
│       └── tlc-configuration.md   # TLC config guide
├── examples/                 # Git submodule: tlaplus/Examples
│   └── specifications/       # 100+ real-world TLA+ specs
└── README.md
```

## TLA+ Examples Repository (Submodule)

This plugin includes the official [TLA+ Examples repository](https://github.com/tlaplus/Examples) as a git submodule in the `examples/` directory. This provides 100+ real-world TLA+ specifications for reference.

### Clone with Submodule
```bash
git clone --recurse-submodules <repo-url>
# Or if already cloned:
git submodule update --init --recursive
```

### Notable Examples in `examples/specifications/`
- `Paxos/` - Paxos consensus algorithm
- `Raft/` - Raft consensus protocol
- `TwoPhase/` - Two-phase commit
- `transaction_commit/` - Transaction protocols
- `ewd840/` - Dijkstra's algorithms
- `lamport_mutex/` - Lamport's mutual exclusion

## Example Specifications in Skills

The plugin's skill documentation includes comprehensive examples based on the TLA+ examples repository:

1. **Key-Value Store** - Snapshot isolation transactions
2. **Bakery Algorithm** - Mutual exclusion
3. **Producer-Consumer** - Bounded buffer synchronization
4. **Echo Algorithm** - Distributed spanning tree
5. **Elevator System** - Multi-car elevator coordination
6. **Cigarette Smokers** - Classic synchronization problem
7. **Consensus Protocol** - Quorum-based agreement
8. **Two-Phase Commit** - Distributed transaction coordination

## TLA+ Resources

- [TLA+ Home Page](https://lamport.azurewebsites.net/tla/tla.html)
- [Learn TLA+](https://learntla.com/)
- [TLA+ Video Course](https://lamport.azurewebsites.net/video/videos.html)
- [TLA+ Examples Repository](https://github.com/tlaplus/Examples)

## Requirements

- Claude Code 1.0.33 or later
- For running TLC: Java Runtime Environment

## License

MIT
