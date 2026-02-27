# Claude Configs

My personal set of Claude Code configurations, including custom commands and agents.
This is simple by purpose. You can turn it into a skill and a plugin, and orchestrate with ticketing systems, MCP servers, and more.

By [Greger Teigre Wedel](https://stuff.greger.io) inspired by works from https://github.com/humanlayer/humanlayer.

## Work process

The work process supports teams and works like this and stores documents in thoughts/:

1. The thought process and work steps live in the repo through a series of files that are checked in.
2. Each major piece of work goes through steps: research, planning, implementation, verification, and iteration.
3. Each step can be connected to sources of context like Linear, web search, etc, but is captured in the doc files.
4. The PR review can be automated or manual, but should always include the thought process, not only the code.

Notes:

- The planning step has two commands: /create_plan and /update_plan. The update_plan is used after having created the first plan and where substantial changes are needed.
- The implementation step also has two commands: /implement_plan and /iterate_plan. The implement_plan command is to implement the plan in phases. You can also have agents work on different phases. The iterate_plan is to make modifications when you review the result. The plan will then be updated to capture the deltas.
- The verification step is a bit of an LLM step (or similar to how you would review the work of a junior). Always assume that something has been missed. The /verify_implementation command creates a verification document that points out missing tasks, bugs, and stuff that was missed when implementing.

## Contents

- **agents/** - Custom agent definitions
- **commands/** - Custom slash commands

## License

This work is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

You are free to use, share, and adapt these configurations for any purpose.
