# Le Data Model de Mistral Vibe (`vibe/core/types.py`)

Documentation de Hafsa pour le groupe **Core** ([issue #6](https://github.com/rrobbes/mistral-vibe/issues/6), [PR #15](https://github.com/rrobbes/mistral-vibe/pull/15)).

## De quoi s'agit-il ?

[`vibe/core/types.py`](../../../../vibe/core/types.py) n'est pas une fonctionnalité qu'on exécute : c'est le **vocabulaire commun du moteur**. Il définit, principalement sous forme de modèles Pydantic, les objets qui circulent entre les backends LLM, la boucle d'agent, les outils, la persistance des sessions et l'App Server.

Question centrale : **comment les données sont-elles représentées, transformées et échangées entre les composants de Mistral Vibe ?**

| Famille | Types principaux | Rôle |
|---|---|---|
| Messages | `LLMMessage`, `Role`, `ToolCall`, `FunctionCall` | un message de la conversation et les appels d'outils demandés par le modèle |
| Streaming | `LLMChunk`, `LLMUsage`, `StopInfo` | les fragments de réponse, additionnables avec `+` |
| Événements | `BaseEvent` et 19 sous-classes | le contrat entre la boucle d'agent et l'App Server |
| Historique | `MessageList` | la conversation, protégée pour un accès concurrent |
| Sessions | `SessionMetadata`, `ChildSessionLink`, `WorktreeContext`, `ScheduledLoop` | ce qui est enregistré sur disque |
| Statistiques | `AgentStats` | tokens, appels d'outils, coût |
| Erreurs | `RateLimitError`, `ContextTooLongError`, `ResponseTooLongError`, `RefusalError` | échecs remontés par les fournisseurs |

## Fichiers de ce dossier

| Fichier | Contenu |
|---|---|
| [`types.md`](types.md) | rapport technique détaillé (en anglais) : architecture, catalogue des types, diagrammes UML Mermaid, déroulement d'un tour, extraits de code commentés, spécifications, tests et observations |
| `README.md` | ce guide de navigation |
| `index.html` | *(à venir)* site interactif utilisé pour la présentation, autonome, à ouvrir directement dans Firefox ou Chromium |

## Par où commencer ?

1. **Vue d'ensemble** : [§1 Summary](types.md#1-summary) et [§2 Architecture](types.md#2-where-the-module-sits-in-the-architecture).
2. **Diagrammes de classes** : [§4 Entity relationships](types.md#4-entity-relationships).
3. **Un tour d'agent pas à pas** : [§5 Life of a turn](types.md#5-life-of-a-turn-seen-through-the-types).
4. **Le streaming** (fusion des fragments avec `+`) : [§6.2](types.md#62-llmmessage__add__--rebuilding-a-streamed-message) et [§6.3](types.md#63-llmchunk__add__-and-llmusage__add__--a-small-monoid).
5. **Qualité et observations** (code mort, champs perdus par `__add__`…) : [§9 Quality, tests and findings](types.md#9-quality-tests-and-findings).

## Liens avec les autres sujets

- **Backends LLM** : [`../backend/`](../backend/), producteurs des `LLMChunk`.
- **Permissions** : [`../../configuration/permissions/`](../../configuration/permissions/), qui s'appuient sur `ApprovalRequestEvent` et `ToolResultEvent.approval_source`.
- **Outils** : [`../../tools/`](../../tools/), qui utilisent `ToolCall`, `ToolCallEvent`, `ToolStreamEvent` et `ToolResultEvent`.
- **Configuration** : [`../../configuration/config-system/`](../../configuration/config-system/), dont un instantané est conservé dans `SessionMetadata.config`.
- **Sessions** : [`../../gestion_sessions/sessions/`](../../gestion_sessions/sessions/), qui persistent `LLMMessage` et `SessionMetadata`.

Décisions d'architecture concernées : [ADR 0002](../../../adr/0002-core-engine-and-delivery-surfaces.md) (moteur et surfaces), [ADR 0003](../../../adr/0003-event-driven-agent-loop.md) (boucle événementielle), [ADR 0006](../../../adr/0006-local-sessions.md) (sessions locales).

## Version étudiée

Le rapport analyse le commit `5e931ff3` (v2.26.1). Aucun fichier Python de `vibe/` n'a été modifié entre ce commit et `origin/main` au 15/11/2026 : l'analyse reste valable.
