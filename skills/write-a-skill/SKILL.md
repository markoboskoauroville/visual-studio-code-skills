---
name: write-a-skill
description: Turn a verified new lesson into or update a reusable VS Code skill, preserving commands, failure signs, checks, and safety rules. Use whenever an interaction teaches a repeatable durable way of working.
---

# Write a skill after every real lesson

Marko asked for one repository that keeps everything learned by Visual Studio Code/Copilot, and specifically requested a
skill for writing every new skill.

## Trigger

Use this skill when a task produces any of these:

1. a command or API sequence that worked and will be needed again;
2. a failure with a proven root cause and a reliable recovery;
3. a safety boundary discovered through real use;
4. a durable location, port, shortcut, UI route, or test method;
5. a correction from Marko that changes future behavior.

Do not write temporary task status, secrets, private tokens, passwords, personal data, or facts that can be rediscovered
cheaply from the code.

## Workflow

1. Search this repository for an existing skill on the same topic.
2. Reuse and update an existing skill before creating a duplicate.
3. Use one narrow job per skill and name the folder in lowercase kebab-case.
4. Create `skills/<name>/SKILL.md` with this frontmatter:

   ```yaml
   ---
   name: <lowercase-kebab-name>
   description: <one sentence saying what the skill does and when to use it>
   ---
   ```

5. Include:
   - **Trigger**: words/events that select the skill;
   - **Space**: machine, app, device, folder, port, or URL;
   - **Procedure**: exact commands in order;
   - **Verification**: the readback proving success;
   - **Safety**: irreversible, noisy, destructive, or privacy-sensitive constraints;
   - **Turning point**: known failures and exact remedies;
   - **History**: newest dated lesson at the top.
6. Quote Marko only when the exact wording changes behavior. Paraphrase technical facts.
7. Verify commands with a real parser, dry-run, or focused test. Do not claim a skill works from prose alone.
8. Check the repository for tokens and private values before publishing.
9. Commit and push the skill in the same completed session.

## Verification template

- Command/status response is exact, not inferred.
- UI state is read back after an action.
- A failure path is explicit and does not use a success-shaped fallback.
- The skill names the cleanup or restore action when the task changes sound, volume, focus, input lock, or a running app.

## History

- 2026-11-26: Marko requested this repository and the meta-skill after repeated learning about Macro Recorder,
  visible Terminal control, and Pixel LTC.
- 2026-11-26: Every skill update must be committed and pushed in the same completed session.
