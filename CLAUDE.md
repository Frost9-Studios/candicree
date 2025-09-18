# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Unity 6.0 LTS game project called **Candicree**, targeting primarily Windows (Steam).  
Game code uses the `Candicree.*` namespace. External, reusable packages live under `Frost9.*` in `/Packages`.

- **Unity version**: 6.0.56f1 (stay on 6.0.x LTS until further notice)
- **Render pipeline**: URP may be added later, but keep code RP-agnostic
- **Input**: Unity New Input System (Rewired may be added if needed)
- **Async**: Prefer UniTask; replace coroutines where reasonable

## Architecture

The project follows a modular, decoupled architecture.  
**Always consult the `README.md` inside each Frost9 package before using it.** These are the authoritative guides and include examples.

### Frost9 Packages
- **Frost9.Audio** — FMOD 2.02 integration, DI-friendly, auto-generated refs
- **Frost9.EventBus** — R3-based event bus, main-thread aware
- **Frost9.HSM** — hierarchical state machine, Unity adapter, event-driven transitions
- **Frost9.Utilities** — timers, adapters, extensions, helpers

### Key External Dependencies
- **FMOD 2.02** — audio middleware (never reference older APIs)
- **VContainer 1.17** — dependency injection
- **R3 1.30** — reactive streams
- **UniTask 2.5.10** — async/await support for Unity
- **ZLinq 1.5.2** — allocation-free LINQ

Other Unity packages (Odin Inspector, Animancer, DOTween Pro, Feel, etc.) may be added as needed. If present, prefer them over reinventing.

## Development Workflow

- **Planning first**: Always produce a short plan (see template below) and wait for approval before coding.
- **Editing boundaries**: Default edits go in `Assets/_Project/_Scripts/`.
  - Do not touch `Assets/Resources/`.
  - Do not edit outside `Assets/` without approval.
- **Docs**: Use **Context7 MCP** (Unity 6.0, VContainer, R3, UniTask, ZLinq) before the web. Only use the web if MCP lacks the docs (e.g., FMOD).
- **XML docs**:
  - `<summary>` at the top of each file's main type
  - XML docs for all public members
  - Code should be self-commenting; inline comments only if necessary

### Planning Template

```text
# Plan
Goal: (one-sentence feature/change)
Context: (relevant Frost9 packages / rules)
Design: (namespaces, events/messages, states, DI, async/timers)
Docs: (XML docs impact)
Files: (exact _Scripts/ paths to edit/create)
```

## Project Structure

```
Assets/
├── _Project/
│   ├── _Scripts/            # main code (default edit target)
│   └── (Prefabs, Materials, etc.)  # created manually
├── Resources/               # do not edit
└── Packages/                # external + Frost9 packages (don't edit directly)
```

Unity manages .meta files and manifest.json automatically — do not modify them.

## Assembly Definitions

Game code assemblies use `Candicree.*` naming (e.g., `Candicree.Gameplay`).

Frost9 packages use their own `Frost9.*` assemblies.

Minimal asmdefs; I will configure them manually.

## Development Notes

**EventBus**: Use for cross-feature communication. Follow README patterns (immutable payloads, domain-first event names).

**HSM**: Subscribe on enter, unsubscribe on exit; transitions via events. Follow README.

**Async**: UniTask for async flows; Frost9.Utilities timers and R3 for scheduling. Pass CancellationToken where appropriate.

**Input**: Unity New Input System; Rewired optional later.

**Performance**: Aim for allocation-free, event-driven design.

## Sub-Agents

Claude Code should be aware of configured sub-agents:

- **unity6-test-expert** — validate correctness
- **unity6-performance-auditor** — flag GC allocations/hot paths
- **unity6-code-reviewer** — ensure Unity 6.0 API usage (not older)
- **unity6-async-expert** — enforce UniTask/R3 patterns
- **messaging-state-architect** — enforce EventBus/HSM messaging rules

## Git Commit Messages

Use conventional commit prefixes:

- `feat`
- `fix`
- `docs`
- `style`
- `refactor`
- `test`
- `chore`

## Example

```csharp
namespace Candicree.Gameplay
{
    /// <summary>
    /// Maintains player health and publishes damage events via Frost9.EventBus.
    /// </summary>
    public sealed class PlayerHealth : UnityEngine.MonoBehaviour
    {
        public void TakeDamage(int amount)
        {
            Frost9.EventBus.EventBus.Publish(new PlayerDamaged(amount));
        }
    }

    /// <summary>Raised when the player takes damage.</summary>
    public readonly struct PlayerDamaged
    {
        public readonly int Amount;
        public PlayerDamaged(int amount) { Amount = amount; }
    }
}
```