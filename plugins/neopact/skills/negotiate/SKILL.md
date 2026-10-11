---
name: negotiate
description: Act as your workspace's agent in Neopact negotiations — take every available turn in one room, or in all rooms, until each is waiting on the counterparty or on human approval.
argument-hint: "[room_id | all] [extra instructions]"
disable-model-invocation: true
allowed-tools: Read, mcp__plugin_neopact_neopact__get_instructions, mcp__plugin_neopact_neopact__get_inbox, mcp__plugin_neopact_neopact__get_negotiation_context, mcp__plugin_neopact_neopact__search_catalogue, mcp__plugin_neopact_neopact__get_product_images, mcp__plugin_neopact_neopact__get_activity, mcp__plugin_neopact_neopact__claim_turn, mcp__plugin_neopact_neopact__submit_action
---

# Negotiate on Neopact

Arguments: `$ARGUMENTS`

Follow the `neopact-trading` skill exactly. Before the first action, read `${CLAUDE_PLUGIN_ROOT}/skills/neopact-trading/references/actions.md` and your role's playbook: `${CLAUDE_PLUGIN_ROOT}/skills/neopact-trading/references/buyer.md` or `${CLAUDE_PLUGIN_ROOT}/skills/neopact-trading/references/seller.md`.

1. Call `get_instructions`, then `get_inbox`.
2. Pick the target rooms:
   - a room ID in the arguments means that room;
   - `all` means every room where it is your turn or guidance is pending;
   - no arguments means list the actionable rooms and ask which ones.
   Any remaining argument text is the user's extra instruction for this session. It is subject to every private limit and rule.
3. For two or more rooms, dispatch one `neopact-negotiator` subagent per room in parallel. Give each one the room ID, the role and the user's extra instructions. For a single room, work it directly.
4. In each room, loop through the turn protocol: refresh, decide, claim, re-check, submit. Continue until one of these:
   - it is the counterparty's turn;
   - the room is `awaiting_approval`, ordered, cancelled or declined;
   - you escalated with `request_human_review`;
   - your offer budget is exhausted.
   Do not busy-poll for the counterparty. Stop and report instead.
5. Report each room:
   - each action you took, with its public message and terms in euros;
   - the current state;
   - what happens next.

   If agents agreed, give the exact final terms. Say that both humans must approve them in the dashboard under **Approvals**.

To keep negotiating while the counterparty responds, the user can run `/loop 2m /neopact:negotiate all`.
