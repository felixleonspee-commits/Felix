---
name: ig-human
description: >-
  Humanize any Instagram draft (caption, script, comment, reply, DM, bio,
  carousel copy) before the user sees it. Runs the user's own `humanizer`
  skill for the actual rewrite, wrapped in a mechanical cleanup of invisible
  characters and typography and a five-check score. Use whenever another ig-*
  skill says to humanize, when the user says humanize, "does this sound like
  AI", "de-slop this", or before any Instagram text is shown to the user.
---

# ig-human

The rewriting is done by the user's own `humanizer` skill
(`anthropic-skills:humanizer`). It knows their style and works in German and
English. The two scripts in this folder only do what a script is good at:
deleting characters nobody can see, and putting a number on the result. When
the scripts and `humanizer` disagree about a word or a sentence, `humanizer`
wins.

## Order of operations

1. **Mechanical pass.** Save the draft to a file and run

   ```bash
   python3 humanize.py draft.txt -o clean.txt --report
   ```

   This strips invisible characters (zero-width spaces, joiners, soft hyphens,
   BOMs, Unicode tags, non-breaking spaces), turns em dashes into commas,
   curly quotes into straight ones and the ellipsis character into three dots.
   It also swaps stock English phrases from `slop.json`. Treat those swaps as
   suggestions: if one changes the meaning or sounds wrong in German, undo it.

2. **The real rewrite.** Invoke the `humanizer` skill with the Skill tool on
   `clean.txt`, together with the structural flags from the `--report`
   output (rule-of-three, "it's not just X, it's Y", staged one-word
   questions, video preambles, emoji bullet lists, shouted words, hashtag
   walls, "follow for more"-style bait). Follow `humanizer`'s instructions in
   full. Keep the meaning, the facts, the CTA and the keyword the user asked
   for. Keep the user's voice from `~/.claude/instagram/voice.md` if it
   exists.

3. **Score it.**

   ```bash
   python3 detect.py draft.txt final.txt
   ```

   Five local heuristics, 0-100, higher is more human: BURSTINESS (sentence
   length variation), SPECIFICITY (numbers, names, concrete markers),
   SLOP DENSITY, FINGERPRINT (invisible chars, em dashes, curly quotes),
   VOICE (contractions, person, structural tells). PASS needs 70+ overall
   with no check under 55.

   The lexicon and the VOICE check are built on English. On German text,
   read SLOP DENSITY and VOICE as rough hints only. FINGERPRINT and
   BURSTINESS work in either language. If the score and `humanizer`'s
   judgement disagree on German text, trust `humanizer`.

4. If FINGERPRINT is not clean, re-run step 1 on the final text. If another
   check is weak, hand that specific problem back to `humanizer` once. Two
   rounds is the limit; past that the draft needs a new angle, not more
   passes.

5. Show the user the final text and the score. Never the score alone.

## Say this honestly

The score is five local heuristics. Nothing is uploaded. They are not
GPTZero, Originality, Copyleaks or Turnitin and cannot promise those verdicts.
Never tell the user a text is undetectable.

## Editing the lexicon

`slop.json` is meant to be edited. If the user has a word they always use and
the script strips it, take it out of the file.
