---
status: empty
iteration: 0
critic_feedback: []
---

<!--
  This file is the communication interface between the research agent and the critic agent.

  RESEARCH AGENT: Write the full draft report below the frontmatter. Update the report
  based on critic feedback. Clear critic_feedback after processing. Only you edit the report body.

  CRITIC AGENT: Read the report body. Independently verify every factual claim via web search.
  Write your findings into the critic_feedback field in the frontmatter. Do NOT edit the report body.

  STATUS VALUES:
    empty       - No report yet
    draft       - Research agent has written the initial report, ready for first review
    in_review   - Critic agent is currently reviewing
    revised     - Research agent has processed feedback and updated the report
    approved    - Critic has approved all sections, loop is complete

  CRITIC FEEDBACK FORMAT:
    - section: "Section Name"
      verdict: "APPROVED" | "ISSUE" | "UNVERIFIABLE"
      details: "Explanation of what's wrong and what the critic found instead" (omit if APPROVED)
      claim: "The exact text that is problematic" (omit if APPROVED)
-->
