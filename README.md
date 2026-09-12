# G-Lore: Game Live Overlay Retrieval Engine

## Project status

This is a planning document. The repository currently contains no application code, tests, or build configuration. It is the starting point for the first implementation.

## Product brief

G-Lore is a game live overlay retrieval engine.

Players often have to research how to play a game by reading wikis, watching YouTube videos and tutorials, and constantly Alt-Tabbing to find the most effective way to play. G-Lore aims to solve that pain point. It is a live AI game companion that assists players while they play.

The product should let a player get help without leaving the game. The player opens an overlay, asks a question, and receives a concise answer with links to the sources behind it. If the player chooses to share a screen region, G-Lore can use that context as well. The first release should prove this workflow for one game before expanding to more games or more automatic game-state understanding.

## UI/UX designs

The following pain points define the UI/UX work for G-Lore. Each one maps to technical requirements that can be tested during implementation.

### Player pain points

🔴 ➜ **Interrupted gameplay:** Players lose focus when they Alt-Tab away from the game. The interruption breaks gameplay and can make them lose track of what they were doing while researching.

🔴 ➜ **Difficult research:** Researching wikis and guides is difficult because players may need to type long questions, verify whether the information is reliable, and account for frequent game updates that can make it outdated.

🔴 ➜ **Hard-to-use overlays:** Some existing game wikis and AI tools are difficult to use. Their overlays can be hard to navigate and may reduce game performance.

🔴 ➜ **Limited screen understanding:** Most existing tools lack OCR and computer vision, so they cannot interpret the information visible on the game screen.

🔴 ➜ **Slow or incomplete interaction:** Players need timely interaction with the companion. Responses must arrive quickly and use enough context to be useful.

🔴 ➜ **Limited game coverage and unclear capability:** The tools should support as many games as possible. Players may expect the system to understand more game state than it can or know information that is missing from the approved sources.

### Technical requirements

<details>
<summary><strong>TR-UX-01: Keep the player in the game</strong></summary>


| Primary requirements | Secondary requirements |
|---|---|
| **Hotkey access:** The desktop client shall open and close the companion through a configurable global hotkey without requiring the player to Alt-Tab.<br><br>**Focus control:** Opening the overlay shall not permanently take input focus away from the game.<br><br>**Return to game:** Closing or cancelling the overlay shall return input focus to the game without restarting or reloading the game.<br><br>**Draft preservation:** The client shall preserve the player's unfinished question when the overlay loses focus or is temporarily dismissed. | **Window modes:** The overlay shall support the game window modes used by the first supported game, including fullscreen or borderless mode where technically possible.<br><br>**Performance measurement:** The client shall record hotkey-to-overlay-visible latency so responsiveness can be tested on the reference machine.<br><br>**Hotkey fallback:** The client shall provide a configurable hotkey when the default binding conflicts with the game. |

</details>

<details>
<summary><strong>TR-UX-02: Reduce the effort of researching reliable, current information</strong></summary>


| Primary requirements | Secondary requirements |
|---|---|
| **Natural-language input:** The client shall accept short natural-language questions without requiring the player to use a specific search syntax.<br><br>**Game scope:** The retrieval service shall return information scoped to the selected game and supported game version.<br><br>**Source traceability:** Every answer shall identify the source passages and provide links to the original sources.<br><br>**Freshness metadata:** Source records shall include freshness and version metadata so outdated information can be identified. | **Ingestion checks:** The knowledge pipeline shall detect duplicate, changed, and unavailable source content during ingestion.<br><br>**Evidence status:** The answer service shall distinguish current evidence from stale or version-uncertain evidence.<br><br>**No-answer state:** The UI shall show when the system cannot find enough reliable information to answer the question. |

</details>

<details>
<summary><strong>TR-UX-03: Make the overlay easy to navigate without harming game performance</strong></summary>


| Primary requirements | Secondary requirements |
|---|---|
| **Consistent layout:** The overlay shall provide a consistent layout for question entry, loading, answer display, citations, errors, and dismissal.<br><br>**Keyboard access:** The main interaction flow shall be usable with the keyboard, including focus order, submission, cancellation, and returning to the game.<br><br>**Safe placement:** The overlay shall avoid covering the game area unnecessarily and shall support repositioning or resizing.<br><br>**Process isolation:** The client shall keep overlay rendering and request handling separate from the game's render and input processes. | **Resource measurement:** The client shall measure CPU, memory, frame-time, and input-impact data while the overlay is open and closed.<br><br>**Readable display:** The overlay shall support readable text scaling and sufficient contrast at the first supported game's target resolutions.<br><br>**Clear states:** The client shall expose distinct UI states for loading, timeout, network failure, unsupported game, and no answer. |

</details>

<details>
<summary><strong>TR-UX-04: Use screen context through OCR and computer vision</strong></summary>


| Primary requirements | Secondary requirements |
|---|---|
| **Region selection:** The client shall let the player select a screen region and preview it before submission.<br><br>**OCR context:** OCR shall be able to extract visible game text from the confirmed region and attach it to the request as context.<br><br>**Context separation:** The context pipeline shall keep OCR text, image data, and typed questions distinguishable in the request payload.<br><br>**Replaceable vision layer:** Computer-vision processing shall be isolated behind a replaceable context interface so providers can change without changing the overlay. | **Explicit consent:** Screen capture and upload shall require an explicit player action and shall not run continuously by default.<br><br>**Capture editing:** The client shall allow the player to crop, remove, or cancel captured content before submission.<br><br>**Graceful fallback:** OCR or computer-vision failure shall leave the typed-question flow usable and shall explain the fallback to the player.<br><br>**Context limits:** The system shall enforce capture size, format, and processing-time limits. |

</details>

<details>
<summary><strong>TR-UX-05: Provide timely, context-aware interaction</strong></summary>


| Primary requirements | Secondary requirements |
|---|---|
| **Progress feedback:** The request pipeline shall provide visible progress from submission through retrieval and answer generation.<br><br>**Cancellation:** The client shall support request cancellation and shall prevent a late response from replacing a newer request.<br><br>**Context metadata:** Each request shall include the selected game, relevant game version, context timestamp, and the context types supplied by the player.<br><br>**Grounded response:** The answer service shall return a concise response that uses the available context and cites the evidence used. | **Latency breakdown:** The system shall measure overlay, OCR, retrieval, model, and total response latency separately.<br><br>**Timeout recovery:** The API shall enforce timeouts and return a recoverable error instead of leaving the overlay in an indefinite loading state.<br><br>**Performance target:** The initial performance target shall be agreed during implementation and tested with representative game questions and screen context. |

</details>

<details>
<summary><strong>TR-UX-06: Scale across games while setting accurate expectations</strong></summary>


| Primary requirements | Secondary requirements |
|---|---|
| **Game profile:** Game-specific configuration shall be represented through a stable game profile containing detection rules, supported versions, and approved sources.<br><br>**Support check:** The system shall identify whether the active game is supported before sending a game-specific request.<br><br>**Capability messages:** Unsupported games, unsupported versions, and missing source coverage shall produce clear capability messages.<br><br>**Evidence boundary:** The answer service shall not present information as game-specific when it cannot trace the information to approved evidence. | **Configuration-based expansion:** Adding a supported game shall require configuration and knowledge-source changes without rewriting the core overlay flow.<br><br>**Visible capability:** The UI shall show the detected game, supported version, and available context capabilities.<br><br>**Manual correction:** The player shall be able to correct or clear an incorrect game selection.<br><br>**Coverage distinction:** The system shall distinguish between a lack of game-state context and a lack of knowledge-base coverage. |

</details>
