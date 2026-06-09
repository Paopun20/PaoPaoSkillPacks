---
name: feedback
description: >
  Design and run conversational feedback sessions for any context — products, events, workshops, services, teams, and more. Use this skill whenever the user wants to collect feedback, run a feedback session, create a feedback survey or form, design interview questions, do a retrospective, gather user opinions, or ask someone structured questions about an experience. Also trigger when the user says things like "let's get feedback on X", "run a feedback session", "I want to survey people about Y", or "help me design feedback questions". This skill handles the full loop: understand the context, design the right questions, run the session interactively, and produce a clean summary.
---
 
# Feedback Skill
 
This skill covers the full feedback lifecycle:
1. **Design** — understand context, generate tailored questions
2. **Run** — conduct a conversational session, one question at a time
3. **Summarize** — produce a structured summary of responses
---
 
## Step 1: Understand the Context
 
Before designing questions, ask the user (the *organizer*) a few quick things. Keep it conversational — don't dump a list of questions at once.
 
Gather:
- **What are you collecting feedback on?** (product, event, service, team, course, etc.)
- **Who will be answering?** (customers, attendees, employees, users, etc.)
- **What do you most want to learn?** (what worked, what didn't, NPS-style sentiment, specific feature opinions, etc.)
- **How many questions?** Default: 5–8 unless they specify. Fewer = higher completion, more = richer data.
- **Tone?** Friendly/casual, professional/neutral, or structured/formal. Default: friendly.
If the context is obvious from the conversation, skip or shorten this step and confirm your assumptions inline.
 
---
 
## Step 2: Design the Questions
 
Generate a set of questions tailored to the context. Follow these principles:
 
### Question mix (adapt to context)
- **1 overall rating** — a single score to anchor sentiment (e.g., "On a scale of 1–10, how would you rate...?")
- **2–3 specific topic questions** — dig into the areas the organizer cares about
- **1 highlight question** — "What was the best part / most useful thing?"
- **1 improvement question** — "What's one thing we could improve?"
- **1 open close** — "Anything else you'd like to share?"
### Question quality rules
- One idea per question — no "and"s
- Prefer open-ended over yes/no (unless doing a quick check-in)
- Avoid leading language ("How great was...?" → "How would you describe...?")
- Match the tone to the audience (casual for internal teams, professional for clients)
### Output format
Present the questions as a numbered list. Briefly explain the purpose of each question to the organizer (one sentence). Then ask: **"Would you like to adjust any of these before we start the session?"**
 
---
 
## Step 3: Run the Session
 
Once the organizer approves the questions, switch into **session mode**. The respondent might be the same person as the organizer, or they might hand off (e.g., "Now I'll answer as the participant").
 
**Session rules:**
- Ask **one question at a time** — never batch them
- After each answer, give a brief, warm acknowledgment (1 sentence max — don't over-praise)
- If an answer is very short or vague, ask one natural follow-up: "Could you tell me a bit more about that?"
- Keep track of all responses internally as you go
- After the final question, thank the respondent and let the organizer know the session is complete
**Opening line** (adapt to tone):
> "Thanks for taking the time! I'll ask you a few questions — answer honestly, there are no right or wrong answers. Ready to start?"
 
---
 
## Step 4: Summarize
 
After the session, produce a **Feedback Summary** with:
 
```
## Feedback Summary
 
**Context:** [what the feedback was about]
**Respondent:** [who answered, if known]
**Date:** [today's date]
 
### Overall Sentiment
[1–2 sentences on the overall rating/vibe]
 
### Key Highlights
- [What worked well, from their answers]
 
### Areas for Improvement
- [What they'd change or found lacking]
 
### Notable Quotes
> "[Direct quote from a meaningful answer]"
 
### Full Responses
1. [Question] → [Answer]
2. [Question] → [Answer]
...
```
 
If multiple people have answered (multi-respondent mode—see below), aggregate the responses.
 
---
 
## Multi-Respondent Mode
 
If the organizer wants to collect feedback from **multiple people**:
 
1. After the session with the first respondent, ask: "Would you like to run another session for a different respondent?"
2. Keep collecting sessions. Track each respondent's answers separately.
3. After all sessions are done, produce an **Aggregated Summary**:
   - Average rating (if applicable)
   - Themes that appeared across multiple responses
   - Notable outliers
   - All individual summaries appended below
---
 
## Tone Guide
 
| Tone | Style |
|------|-------|
| Friendly/casual | Warm, conversational, use "you", light contractions |
| Professional/neutral | Clear, respectful, no slang, measured |
| Structured/formal | Precise, numbered, no filler phrases |
 
Default to **friendly/casual** unless told otherwise.
 
---
 
## Edge Cases
 
- **Organizer wants a shareable form instead of a live session**: Generate the questions as a clean copy they can paste into Google Forms, Typeform, or a similar tool. Format each question with its type (short answer, multiple choice, scale 1–10, etc.).
- **Very short feedback ("just one question")**: Skip the design step, ask the single question, record the answer, done.
- **Sensitive topics** (HR, complaints, anonymous feedback): Note that Claude.ai conversations aren't anonymous, and suggest they use a proper anonymous feedback tool if confidentiality matters.
 
