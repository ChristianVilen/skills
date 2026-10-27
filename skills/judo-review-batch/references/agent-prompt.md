# Agent prompt

Fill in the values in `{}`. Put the notes from Step 2 at `{notes}`. If the PR
has no notes, delete that line.

```
Review PR #{n} in {repo}: "{title}" by @{author}.
The PR belongs to someone else.

1. Read ~/.claude/skills/judo-code-review/SKILL.md and do
   the review that it describes. You cannot start it with the Skill tool, so
   follow the file.
   Ownership: someone else's (@{author}). Skip the ownership checks.
2. Scope: only the diff of this PR against its base branch `{base}`.
   {notes}
   You are in your own worktree. Fetch the base branch and pull/{n}/head, then
   check out the head commit (detached HEAD). Read files at that commit, so the
   line numbers are correct. If the head moves while you work, review the new
   head and tell me.
3. Read only. Do not change source files, install packages or run tests.
4. Do not ask the "Next actions" question. Go directly to "Create pending
   GitHub review", as the skill describes it. Load the unslop skill before you
   write the comments. Use plain words and short sentences. Say what the problem
   is and ask about it. Do not sketch the fix.
   - If I already have a pending review on this PR, do not post. Tell me.
   - Read my reviews that I already submitted on this PR. Do not repeat a
     point that I already made.
   - Never submit the review: send no `event` field.
   - If the review has no findings, do not post an empty review. Tell me.
5. Do not name or link private sources in anything that you post: internal
   knowledge bases, personal notes, or other repos that the readers of this
   PR cannot see.
6. Write the full report (verdict header and findings) to
   {report_dir}/judo-{n}.md.
7. Give back at most 5 lines: verdict, number of findings for each severity,
   the pending review URL (or why you did not post one), and the report path.
```
