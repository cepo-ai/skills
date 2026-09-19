---
name: codebase-trivia
description: Run an interactive multiple-choice quiz about the user's current codebase to refresh their knowledge of its architecture, behavior, and recent changes. Use when the user asks for codebase trivia, a repository quiz, or to test their familiarity with a project.
---

# Codebase trivia

Help the user stay familiar with their project through multiple-choice questions,
asked one at a time. Default to five questions, allow a custom count, and explain
each answer using evidence from the code. Offer an optional Markdown record.

## Session options

Read options from the invocation or conversation, without requiring exact flag
syntax. For example: "10 questions about authentication, beginner, save to
docs/auth-quiz.md."

- **Count:** Five by default; accept any positive integer the user requests.
  Clarify invalid or conflicting counts. Allow a change during the session; if the
  new total has already been reached, finish with the completed results.
- **Topic and difficulty:** Honor the user's choices. Default to moderate
  difficulty and a mix of architecture, behavior, and recent changes.
- **Markdown export:** Off unless requested. Accept "save as Markdown" or a
  destination path at the start, during the quiz, or at the end. A request at the
  start schedules an export when the session ends, including an early stop.
  An explicit request to save current progress exports only progress so far.

State the chosen count and scope briefly, including export details if requested.
Do not make the user answer a setup questionnaire to start with defaults.

## Prepare

- Use the user's current project or explicitly named repository, not the directory
  where this skill is installed. If the target is ambiguous or inaccessible, ask
  for its location before writing questions.
- Follow the repository's instructions. Inspect its structure, entry points,
  configuration, and a few relevant implementations and tests. Use targeted
  searches and reads; don't exhaustively read a large repository before starting.
- If Git history is available, inspect a small set of recent relevant commits and
  their diffs. Check the current implementation before making claims about present
  behavior. Identify a commit explicitly when a question asks about past behavior.
  If history is missing or uninformative, use architecture and behavior questions
  and briefly mention the adjusted scope.
- Inspect read-only; write only an explicitly requested Markdown export. A quiz
  does not require installing dependencies, executing project code, or changing
  branches. Never turn secret values or private credentials encountered in the
  repository into quiz material.

## Build each question

- Ground the correct answer in files you have actually inspected. Record the
  supporting paths and line numbers in session context for the later explanation.
  Documentation can guide exploration; verify implementation claims against code.
- Ask about useful knowledge: component responsibilities, request or data flow,
  configuration effects, error handling, tests of an important contract, or a
  meaningful behavior change. Avoid line-number trivia, obscure spelling, and
  generic programming questions that don't depend on this repository.
- Give three or four plausible choices with exactly one supported answer. State
  any configuration, inputs, or other conditions needed to disambiguate behavior.
  Avoid overlapping options, trick wording, and "all of the above." Vary the
  correct answer's position without making length or phrasing reveal it.
- Prefer a different subsystem or concept for each question. Adjust later
  difficulty to the user's answers and requests. Use only this session's history
  unless the user supplies previous results.
- Verify enough evidence for the next question before asking it. If the repository
  cannot support the requested number of useful distinct questions, offer fewer;
  do not fill the session with invented facts.

## Use the harness's question tool

Inspect the tools exposed by the active host and use its native question/answer or
user-input tool for every quiz question when that tool is available and permitted
in the current mode. This applies across harnesses; do not assume a specific tool
name or an API from another host. Prefer the tool that supports selectable options.
Adapt the number of choices and their labels to its schema while retaining one
question per interaction. If it supports only text input, put the lettered choices
in the question and collect the answer there.

Use the same interface for necessary clarifications and the optional export choice.
Keep choices neutral and do not mark one as recommended or correct. If no permitted
question tool is available, or its constraints cannot present a neutral quiz, say
so briefly and ask in chat. Do not change modes just to obtain a tool or pretend
that a tool was used. A tool failure may also require this chat fallback.

## Ask, wait, explain

1. Show the question number and total. Present one question with lettered options.
   Send it through the question tool selected above, or use the chat fallback.
2. Wait for the user's submitted answer. A preselected option, an unanswered tool
   call, or elapsed time is not an answer. Do not reveal the solution, its source
   excerpt, or later questions while waiting.
3. Accept a letter, option text, or an unambiguous equivalent. Clarify ambiguous
   replies before grading. A request for a hint keeps the question open; give a
   small clue and wait again. On "skip" or "I don't know," explain the answer and
   record a skip. On "stop," end the quiz and summarize progress so far.
4. After an answer, say whether it is correct, identify the correct choice, and
   explain why in a few sentences. Link to the relevant repository files with
   verified line numbers using the host's supported link format. For a wrong
   answer, explain the distinction that made the selected option incorrect.
5. If the user challenges an answer, recheck the evidence. Correct a mistaken grade.
   Withdraw and replace an ambiguous or unsupported question without penalizing
   the user. Don't defend an answer key that the code doesn't support.
6. Record the result in session context, then ask the next question. Keep at most
   one unanswered question active. Retain the question, options, user answer,
   outcome, hints, explanation, and evidence for a possible export. Persist them
   only when the user requests that export.

## Finish

After five questions, or the requested count, give the number correct out of the
number answered, plus any skips and hints used. Mention the concepts the user knew
well and suggest one or two files or concepts to revisit based on missed answers.
If they stopped early, report only the questions completed. Offer another round
or a focused follow-up without automatically starting it. If an export was
requested, save it now. Otherwise, offer to save the session as Markdown using
the question tool; a missing response is not permission to write a file.

## Save as Markdown

- Save to the requested path, resolving relative paths from the project root. If
  no path was specified, use `codebase-trivia-YYYYMMDD-HHMMSS.md` in that root with
  the actual current timestamp. Avoid overwriting an existing file by choosing a
  unique suffix; if an explicit path already exists, clarify whether to replace
  it or use a new name. An export request authorizes creating a new file without
  another confirmation, subject to the host's filesystem permissions.
- Include the project, session date, topic, difficulty, requested count, completed
  count, score, skips, and hints. Include the inspected Git revision when available
  and note if the quiz reflects uncommitted changes; don't invent revision data.
- For each completed question, include its text, choices, the user's answer or
  skip, the correct answer, explanation, and source paths with line numbers. Use
  relative Markdown links from the export file to repository files when possible.
  Finish with the suggested review topics. Keep withdrawn questions out of the
  score and identify corrections if they matter to the record.
- An export made before the session ends must not reveal answers to unasked or
  unanswered questions. Include only completed questions and mark it as a partial
  session. Keep an active question pending after saving progress.
- After writing, verify the file exists with the expected content and provide a
  link to it. Do not stage or commit the export automatically. If file writing is
  unavailable, provide the Markdown in chat for the user to save and state that no
  file was created.
