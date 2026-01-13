# bc-gitops-demo-repo
[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-support-yellow.svg)](https://buymeacoffee.com/beamologist)

GitOps specification repository for [bc_gitops](https://github.com/beam-campus/bc-gitops) demonstration.

## Overview

This repository is watched by `bc_gitops` running inside `demo_web`. When you add, modify, or remove application specifications in the `apps/` directory, bc_gitops automatically reconciles the running system.

## Structure

```
apps/
├── demo_counter/
│   └── app.config    # Erlang counter application
└── demo_tui/
    └── app.config    # Rust TUI application
```

## How It Works

1. `demo_web` (Phoenix app) starts with `bc_gitops` as a dependency
2. `bc_gitops` clones this repository and watches for changes
3. When you push changes to `apps/`, bc_gitops:
   - Detects new apps → deploys them
   - Detects version changes → upgrades (with hot code reload!)
   - Detects removed apps → stops them

## Quick Start

### Deploy demo_counter

Create `apps/demo_counter/app.config`:

```erlang
#{
    name => demo_counter,
    version => <<"1.0.0">>,
    source => #{type => hex},
    env => #{
        port => 8081,
        initial_count => 0
    },
    health => #{
        type => http,
        port => 8081,
        path => <<"/health">>
    },
    depends_on => []
}.
```

Commit and push:

```bash
git add apps/demo_counter/app.config
git commit -m "Deploy demo_counter v1.0.0"
git push
```

Watch the `demo_web` dashboard - demo_counter will appear and start running!

### Upgrade demo_counter

Change the version in `app.config`:

```erlang
version => <<"1.1.0">>,
```

Commit and push. bc_gitops will perform a hot code upgrade, preserving the counter's state.

### Remove demo_counter

```bash
git rm -r apps/demo_counter
git commit -m "Remove demo_counter"
git push
```

## Application Specification Format

```erlang
#{
    name => atom(),           % OTP application name
    version => binary(),      % Version string
    source => #{
        type => hex | git,
        url => binary(),      % For git only
        ref => binary()       % For git: branch/tag/commit
    },
    env => map(),             % Application environment
    health => #{
        type => http | tcp,
        port => integer(),
        path => binary()      % For http only
    },
    depends_on => [atom()]    % Other managed apps that must start first
}.
```

## Related Repositories

- [bc-gitops](https://github.com/beam-campus/bc-gitops) - The GitOps library
- [bc-gitops-demo-web](https://github.com/beam-campus/bc-gitops-demo-web) - Phoenix host app
- [bc-gitops-demo-counter](https://github.com/beam-campus/bc-gitops-demo-counter) - Erlang counter app
- [bc-gitops-demo-tui](https://github.com/beam-campus/bc-gitops-demo-tui) - Rust TUI app

## License

MIT
