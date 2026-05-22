# Behavioral Interview Guide

Technical skills get you the interview. Soft skills get you the offer. Senior roles especially evaluate leadership, communication, and judgment.

---

## The STAR Method

Structure every behavioral answer with STAR:

- **S**ituation: Context and background
- **T**ask: What was your responsibility?
- **A**ction: What did YOU specifically do? (use "I", not "we")
- **R**esult: Quantify the outcome if possible

> ⚠️ Most engineers make these mistakes:
> - Too much Situation, not enough Action
> - Saying "we" — interviewers want YOUR contribution
> - No quantified Result ("it improved performance" → "reduced launch time by 40%")

---

## Common Questions & Answer Framework

### "Tell me about yourself."

**Structure:** Current role → key experience → why this role

> "I'm currently a senior iOS engineer at [Company], where I led the architecture migration from MVC to MVVM for a team of 8 engineers on our 3M-user social app. Before that, I spent 2 years at [Previous], where I built the real-time messaging feature from scratch. I'm looking for this role because I want to [specific reason — technical challenge, company mission, scale]."

**Avoid:** Reciting your resume. Career chronology starting from college. Vague generalities.

---

### "Tell me about a challenging technical problem you solved."

**Framework:** Choose a problem that shows depth. Emphasize:
1. The problem was non-obvious
2. You diagnosed it properly (not by guessing)
3. You considered multiple solutions
4. You chose the right one with justified tradeoffs
5. Measurable outcome

**Example structure:**

> "We had a race condition in our token refresh flow. [Situation] Multiple concurrent API requests were all getting 401 responses and each trying to refresh the token simultaneously, causing token invalidation loops. [Task] I was responsible for our networking layer. [Action] I diagnosed it with logging showing multiple simultaneous refresh calls. I researched the pattern and designed an actor-based solution where the first refresh starts a Task and subsequent callers await the same Task instead of starting new ones. I also added integration tests that deliberately triggered concurrent 401s to verify the fix. [Result] Eliminated all token-loop user reports. The solution is now our pattern for any similar serialization problem."

---

### "Tell me about a time you disagreed with a technical decision."

**What they're looking for:** Professional communication, ability to advocate with data, knowing when to commit.

**Framework:**
1. Respectfully challenged with data/reasoning, not opinion
2. Sought to understand the other perspective
3. Either persuaded OR committed to the team decision after raising concerns
4. Focus on professionalism

> "My team wanted to use a third-party analytics SDK that I had concerns about — it was adding 30ms to our launch time and collecting more data than we needed. I raised this in design review with specific measurements from Instruments. The PM needed the features quickly, so after discussing, we agreed to use it temporarily with a plan to evaluate our own lightweight implementation in Q3. I committed to the decision and helped implement it cleanly. When Q3 came, I led the migration and we cut launch time by 28ms."

---

### "Describe a time you mentored or helped a junior engineer grow."

**What they're looking for:** Leadership signal. Patience, communication, investment in team.

> "A junior engineer was struggling with understanding Core Data threading. Rather than just fixing their crash, I sat with them for 30 minutes and drew the context/thread model on a whiteboard. I then created a short internal document with our team's Core Data patterns and common pitfalls. That engineer became our team's Core Data go-to person within 6 months."

---

### "Tell me about a time you had to make a decision with incomplete information."

**Framework:** Show pragmatic decision-making under uncertainty. Risk assessment. Reversibility.

> "We needed to choose a third-party chat SDK vs building our own 2 weeks before a deadline. I didn't have full performance data for the SDK. I evaluated based on: the team's familiarity with the SDK, the reversibility of the choice (we could swap later), and the known risk of building our own under deadline pressure. We went with the SDK, shipped on time, and 6 months later did a controlled migration to our own implementation when we had the data to justify it."

---

### "What's your greatest technical weakness?"

**What they're looking for:** Self-awareness. Active improvement.

**Good weaknesses:**
- Real, but not disqualifying for the role
- Show awareness and active mitigation

> "I tend to optimize too early. When I see a potential performance issue, I want to fix it immediately, even before we know if it matters. I've learned to resist that impulse — I now always measure first with Instruments and only optimize when I have data showing it's actually a problem. It's made me a better engineer and more valuable to the team because I don't waste time on imaginary bottlenecks."

---

### "Why are you leaving your current role?"

**Good reasons:** Want to work on X type of problem, grow in Y direction, excited about this company's Z mission.

**Avoid:** Complaining about current employer, mentioning money as the primary driver (even if true).

> "I've accomplished my main goals at my current company — I rebuilt our networking layer, introduced MVVM, and grew from mid to senior. I'm looking for a role where I can take on more architectural responsibility across a larger system, which is why [Target Company]'s scale and the [specific product challenge] particularly excite me."

---

## Senior-Level Behavioral Signals

At senior/staff level, interviewers look for these additional signals:

| Signal | What to Demonstrate |
|--------|---------------------|
| Technical leadership | Drove decisions, not just implemented them |
| Code quality ownership | Established patterns, code reviews, documentation |
| Cross-team collaboration | Worked with backend, PM, design, QA effectively |
| Mentorship | Helped others grow |
| Pragmatism | Ship vs perfect; know when each is right |
| System thinking | Thought about how changes affect the whole system |

---

## Questions to Ask the Interviewer

These signal that you're thoughtful about where you work:

**Technical:**
- "What's the biggest technical debt you're currently tackling?"
- "How do you approach architecture decisions? Is there a design review process?"
- "How much of your codebase has test coverage? What's the team's approach to testing?"
- "What's your deployment process? How often do you release?"

**Team & Process:**
- "How does the iOS team collaborate with backend engineers?"
- "What does a typical week look like for an iOS engineer here?"
- "How is technical mentorship structured for engineers at my level?"

**Growth:**
- "What does success look like in the first 90 days for this role?"
- "Where do you see this role evolving in the next year?"
- "What's the learning/conference/education budget like?"

**Avoid:** Asking about vacation, salary (save for offer stage), or questions easily found on the company website.

---

## Final Checklist

### Night Before

- [ ] Review your two or three strongest technical stories (STAR format)
- [ ] Review the company's app — use it, note what you'd improve
- [ ] Re-read the job description — map your experience to each bullet
- [ ] Prepare three specific reasons you want THIS role at THIS company
- [ ] Get good sleep

### Day Of

- [ ] Have examples for: challenge overcome, disagreement, impact, failure/learning
- [ ] Know your numbers: team size, user scale, performance improvements
- [ ] Slow down when nervous — thoughtful pace beats rushing
- [ ] It's okay to say "let me think about that for a moment"
- [ ] Ask clarifying questions before diving into coding problems

### After

- [ ] Send a thank you email within 24 hours (brief, genuine, reference something specific from the conversation)
- [ ] Reflect on what went well and what to improve
- [ ] Follow up if you haven't heard within the stated timeline
