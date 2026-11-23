# SOS QUIZ — Comprehensive Bug Fix & Quality Improvement Plan

## Mission
Audit, repair, and improve the existing SOS QUIZ platform without removing, replacing, disabling, or simplifying any existing feature or capability.

This is a **maintenance and quality-improvement task**, not a redesign or rebuild. Preserve the existing product, UI identity, user flows, data structures, integrations, and features unless a change is strictly required to fix a bug. If a change risks breaking or removing a feature, stop and report the risk before making it.

## Non-negotiable preservation rules
1. Do not delete features, pages, controls, integrations, settings, or user-facing capabilities.
2. Do not rewrite the application from scratch or replace the current implementation with a smaller prototype.
3. Preserve the SOS visual identity, responsive behavior, Arabic RTL support, dark/light themes, and existing navigation.
4. Preserve all current quiz modes, file formats, text extraction, question-type options, AI mode, traditional mode, quiz timer, scoring, result history, XP/levels/streaks, profiles, rooms, chat, leaderboard, reports, and Supabase functionality.
5. Do not delete, reset, migrate destructively, or overwrite user data. If a schema change is needed, make it backward-compatible and provide a safe migration plan.
6. Do not remove a feature merely because it is currently buggy. Fix it.
7. Before changing code, inspect the complete current codebase and identify dependencies and existing behaviors. Avoid assumptions based on this instruction file alone.
8. Keep changes focused, readable, and compatible with the current project structure.
9. Do not claim a feature was tested unless you actually ran the test and observed its result.
10. If a requested fix cannot be safely completed without credentials, database policy access, or external configuration, implement what can safely be implemented and clearly document the remaining step. Do not invent secrets or configuration values.

## Phase 1 — Baseline audit
Before editing:
- Review all source files, configuration, scripts, database-related files, and current tests.
- Create a checklist of all currently implemented features and the files/functions they depend on.
- Identify bugs, likely regressions, security concerns, and weak validation.
- Record the current build/test status before making changes.
- Keep a short backup or use version control so changes can be reverted.
- Do not make unrelated refactors.

## Phase 2 — Quiz question quality (highest priority)

### Traditional generator
Improve the traditional/offline question generator so that it produces questions that are meaningful, answerable, and grounded in the uploaded lecture—not merely random blanks from frequent words.

Requirements:
- Keep the traditional generator available. Do not remove it or make AI mandatory.
- Filter page headers/footers, lecture titles, author names, cover information, course metadata, page numbers, isolated fragments, navigation text, and non-instructional content more reliably.
- Prefer sentences and paragraphs that explain a concept, define a term, describe a process, compare ideas, or state a factual relationship.
- Avoid generating questions from incomplete sentences, broken extraction fragments, or sentences that have no clear educational meaning.
- Do not create a question if the answer cannot be determined confidently from the source text.
- Improve distractors: make them plausible but clearly incorrect according to the source and avoid duplicate options, the correct answer appearing twice, or nonsensical choices.
- Avoid blanking arbitrary frequent words (such as “the”, “is”, “and”, “of”, or similarly low-information words) as the main question-generation strategy.
- Avoid generating multiple near-identical questions from the same sentence or concept.
- Maintain the existing question types and options: MCQ, true/false, and fill-in-the-blank, wherever the user selected them.
- Respect the user's requested number of questions, but never fill the count with low-quality or off-topic items. If the source does not support the requested quantity, explain that fewer valid questions could be generated.
- Make question creation deterministic enough to debug where possible; randomization should not make the quiz quality unpredictable.
- Use the lecture as the sole source of truth for lecture-specific facts. Do not inject unrelated facts from general knowledge.
- Keep Arabic and English lecture content supported.

### AI question generation
Improve the AI prompt and validation pipeline:
- Tell the AI to use only the uploaded lecture content and not add outside facts.
- Exclude titles, author names, cover metadata, page numbers, and irrelevant text.
- Ask for clear, direct, student-appropriate questions that test understanding rather than trivia about document formatting.
- Require answer correctness to be supported by the source.
- Validate the returned JSON structure and each question before rendering.
- Validate question text, answer, type, option count, option uniqueness, and whether the correct answer is represented exactly once.
- For true/false, normalize accepted answer forms without changing the meaning.
- For fill-in questions, ensure the expected answer is present and not excessively ambiguous.
- Reject duplicates, malformed questions, unsupported claims, empty values, and questions that are not grounded in the source.
- If AI generation fails or returns invalid data, use the existing safe fallback without breaking the quiz flow.
- Do not silently accept invalid output just because it parses as JSON.

### Question review and fallback behavior
- Add or improve a quality-check step before questions are shown.
- If fewer valid questions exist than requested, show an honest message and proceed with the valid ones or let the user retry.
- Avoid empty quizzes and avoid crashes when the source text is short or poorly extracted.
- Make error messages helpful and understandable in Arabic.

## Phase 3 — Lecture extraction and text cleanup
Preserve existing supported formats (PDF, DOCX, PPTX, TXT, MD, and pasted text, subject to the current implementation).

- Fix extraction issues without removing file-format support.
- Handle empty files, unsupported files, damaged files, and extraction failures gracefully.
- Keep page markers and page-skip behavior correct.
- Ensure skip-page controls do not accidentally remove all content or create invalid indexes.
- Improve metadata filtering without discarding actual lecture content.
- Preserve the existing Arabic correction tools (character reversal/word-order correction) and make their behavior clear.
- Do not silently alter user text in ways that change the meaning.
- Add checks that extracted content is non-empty and useful before generating questions.

## Phase 4 — Quiz flow, timer, scoring, and repeat-submission protection
- Verify that the timer starts, displays, and counts down correctly.
- When time expires, submit/finish the quiz exactly once and prevent further answer edits for that attempt.
- Ensure manual submission and time-expired submission cannot both process the same attempt.
- Add an explicit submitted/completed state and guard against duplicate submissions.
- Prevent duplicate score records, XP awards, streak changes, or leaderboard submissions from repeated clicks or repeated handlers.
- Preserve the existing result screen, history, statistics, and reward logic; repair them rather than removing them.
- Validate answer normalization carefully so accepted variations work without falsely marking unrelated answers correct.
- Check boundary cases: zero-minute/no-timer mode, very short quizzes, unanswered questions, skipped questions, refresh behavior, and timer expiration.
- Do not change the scoring system or grade thresholds without documenting the issue and asking for confirmation if the change affects user-visible grading.

## Phase 5 — Results, history, and gamification
- Verify result calculation, percentage, correct/incorrect/unanswered counts, grade display, and explanations.
- Review grade labels and threshold consistency. If the current grade mapping appears unusual, flag it and propose a correction before changing the visible grading policy.
- Ensure each attempt is recorded once.
- Ensure history and statistics update consistently.
- Preserve XP, levels, streaks, celebrations, and related features.
- Prevent duplicated XP or streak updates from retries, repeated clicks, or page-state bugs.
- Handle missing or corrupted local data without crashing. Prefer safe recovery over destructive resets.

## Phase 6 — API key and AI integration safety
- Review how AI API keys are collected, stored, used, and cleared.
- Do not expose any secret key in source code, committed files, logs, error messages, or public repository history.
- If the current browser-side key storage/use is retained for development, clearly explain the risk and do not imply it is production-safe.
- Prefer a secure server-side or Supabase Edge Function proxy for public production use, with secrets stored server-side and appropriate authentication, rate limits, and usage controls.
- Do not invent or insert API keys.
- Do not break the existing AI feature while planning a safer architecture. If a backend change requires secrets or deployment settings unavailable to you, prepare the code/configuration and document the exact setup steps without falsely claiming completion.

## Phase 7 — Supabase, accounts, rooms, chat, leaderboard, and reports
Inspect both client implementation and available database/configuration files.

- Preserve login/authentication, profiles, rooms, chat/realtime, leaderboard, and reporting features.
- Validate user-provided data (nickname, profile fields, messages, quiz/result payloads) at appropriate boundaries.
- Check that user content is escaped/safely rendered and that no unsafe HTML injection path is introduced.
- Review room/profile rules, duplicate account/profile handling, and error states.
- Review access control, RLS policies, database functions, realtime permissions, and server-side validation where SQL/configuration is available.
- Do not assume the client alone is a security boundary.
- If database policies are not available in the repository, explicitly mark them as **not audited** and provide the exact policies/files needed for a follow-up audit.
- Keep reporting and moderation affordances. Do not remove reports or chat to “solve” safety issues.
- Add user-friendly handling for connection failures, unavailable services, unauthorized actions, and empty states.
- Do not expose phone numbers, external links, or other restricted chat content if existing product rules prohibit them; enforce rules server-side when possible, not only in the UI.
- Preserve user data and make any necessary schema changes non-destructive.

## Phase 8 — Profile and settings usability
- Review the current profile setup and editing experience.
- If profile details are locked after creation, implement a safe editing flow if compatible with current architecture.
- Preserve existing room matching rules and clearly warn users if changing a field affects their room membership.
- Do not silently change a user's profile or room assignment.
- Keep current profile fields and validations.

## Phase 9 — UX, accessibility, and stability
- Preserve existing design and improve only where a bug or usability issue exists.
- Check responsive layouts at phone, tablet, and desktop widths.
- Check Arabic RTL layout, mixed Arabic/English text, long names, long questions/options, and overflow.
- Check buttons, loading indicators, disabled states, empty states, errors, and success messages.
- Keep controls keyboard accessible and provide meaningful labels where practical.
- Prevent UI freeze during large file processing; use existing async approaches and show progress/loading state where feasible.
- Do not add heavy dependencies without a clear need.
- Ensure no console errors or unhandled promise rejections remain in the tested flows.

## Phase 10 — Test plan (must be performed)
Run the existing test suite and build before and after changes. Add or update tests as appropriate. Report actual outcomes.

Test cases:
1. Upload/paste a clean lecture and generate a traditional quiz.
2. Generate an AI quiz (if credentials/configuration are available).
3. Verify questions stay on lecture content and exclude title/author/metadata.
4. Verify MCQ options are unique and correct answer occurs exactly once.
5. Verify true/false and fill-in formats.
6. Test a short lecture and a noisy/poorly extracted lecture.
7. Test PDF, DOCX, PPTX, TXT, MD, and pasted text, where supported.
8. Test page skipping and Arabic text correction controls.
9. Test manual submit and timer expiry; confirm only one completion occurs.
10. Try repeat clicks/repeated submit; confirm no duplicate attempt, XP, streak, or leaderboard entry.
11. Verify unanswered questions, score, percentage, grade, and history.
12. Verify profile editing (if implemented) and room behavior.
13. Verify chat, realtime, leaderboard, and reporting in the configured environment.
14. Test no-network/failed-network, missing config, invalid file, and malformed AI response.
15. Test responsive layout, RTL, and dark/light themes.

## Phase 11 — Work strategy
- Make a small, logical set of changes at a time.
- Prioritize quiz correctness and prevention of duplicate submissions.
- After each major change, run tests/build and inspect the result.
- Fix regressions caused by your changes.
- Do not delete working code simply to make tests pass.
- Do not “mock away” or disable features to hide failures.
- If a test cannot run, say why and list what is needed.

## Definition of done
The work is complete only when:
- Existing feature inventory is preserved.
- Build succeeds.
- Existing tests pass, and new tests for repaired behavior pass.
- Traditional questions are demonstrably more relevant and source-grounded.
- AI questions are validated before display.
- Timer expiry and manual submission cannot produce duplicate completion.
- Scoring/history/XP updates are consistent and non-duplicated.
- Error handling is improved for file extraction and service failures.
- Security concerns are either addressed or explicitly documented with a practical next step.
- No existing feature has been removed or silently disabled.

## Final report required
At the end, provide:
1. A list of issues found.
2. A list of changes made, grouped by feature.
3. The full list of features confirmed preserved.
4. Test/build commands run and their **actual** results.
5. Tests that could not run and why.
6. Any manual Supabase/OpenAI/deployment steps still required.
7. Any risks or known limitations.

**Do not say “everything is fixed” unless every applicable item was actually implemented and tested.**
