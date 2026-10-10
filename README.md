# handoff-skill

A Hermes Agent skill that keeps durable `HANDOFF.md` checkpoints so another model or session can verify the current state and continue without restarting completed work.

## Usage

Tell Hermes:

> Use the handoff-skill. Always prepare for my tokens running out or a model switch.

A replacement model should be told:

> Use the handoff-skill, read `HANDOFF.md`, verify the current state, and continue from the exact next step. Do not restart completed work.

## Install

Copy this repository into your active Hermes profile under `skills/productivity/handoff-skill/`, then start a new session so the skill catalog reloads.

## Limitation

This skill checkpoints when the agent can execute tools. A strict independent three-minute heartbeat during blocked provider/tool calls requires a separate hook or background writer.
