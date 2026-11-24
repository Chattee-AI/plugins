---
description: Build a web app on Chattee from your description; nothing is built until you approve the plan.
argument-hint: "<what to build>"
---

Build a web app on Chattee for the user with the Chattee MCP tools, following the Chattee skill and the server's own rules.

The user's request: $ARGUMENTS

## Steps

1. If there is no request, ask the user what they want built, and stop until they answer.
2. Call `create_project` with the request in the user's own words as the `prompt`; attach their files with `attachments`. Never add requirements they did not state.
3. Call `wait_for_run` with the `chat_uid` it returned, again and again, until it returns a decision point. A build takes 5–90 minutes; a wait that times out just means call it again.
4. `awaiting_input`: relay the question and its options verbatim, then send the user's answer with `answer_question`. Never answer for them.
5. `awaiting_approval`: show the user the FULL plan and the builder link, and ask for their decision. Call `approve_plan` only after an explicit yes; otherwise call `refine_plan` with their feedback, and wait again.
6. `completed`: relay the closing summary and the Draft preview link. Publish only when the user asks for it.

Money is the user's decision: at `paused_budget`, `payment_required` or `insufficient_credits`, relay the credits used and the builder link, and stop. Continue a paused run only with a total the user names.
