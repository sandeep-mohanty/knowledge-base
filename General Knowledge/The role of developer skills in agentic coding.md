# The Role of Developer Skills in Agentic Coding

As agentic coding assistants become more capable, reactions vary widely. Some extrapolate from recent advancements and claim, “In a year, we won’t need developers anymore.” Others raise concerns about the quality of AI-generated code and the challenges of preparing junior developers for this changing landscape.

In the past few months, I have regularly used the agentic modes in Cursor, Windsurf, and Cline, almost exclusively for changing existing codebases (as opposed to creating Tic Tac Toe from scratch). I am overall very impressed by the recent progress in IDE integration and how those integrations massively boost the way in which the tools can assist me. They:

- Execute tests and other development tasks, and try to immediately fix the errors that occur.
- Automatically pick up on and try to fix linting and compile errors.
- Can do web research.
- Some even have browser preview integration, to pick up on console errors or check DOM elements.

All of this has led to impressive collaboration sessions with AI, and sometimes helps me build features and figure out problems in record time.

**However.**

Even in those successful sessions, I intervened, corrected, and steered all the time. And often I decided not to even commit the changes. In this memo, I will list concrete examples of that steering, to illustrate what role the experience and skills of a developer play in this “supervised agent” mode. These examples show that while the advancements have been impressive, we’re still far away from AI writing code autonomously for non-trivial tasks. They also give ideas of the types of skills that developers will still have to apply for the foreseeable future. Those are the skills we have to preserve and train for.

---

### Where I’ve Had to Steer

I want to preface this by saying that AI tools are categorically and always bad at the things that I’m listing. Some of the examples can even be easily mitigated with additional prompting or custom rules. Mitigated, but not fully controlled: LLMs frequently don’t listen to the letter of the prompt. The longer a coding session gets, the more hit-and-miss it becomes. So the things I’m listing absolutely have a non-negligible probability of happening, regardless of the rigor in prompting, or the number of context providers integrated into the coding assistant.

I am categorising my examples into 3 types of impact radius, AI missteps that:

a. Slowed down my speed of development and time to commit instead of speeding it up (compared to unassisted coding), or  
b. Create friction for the team flow in that iteration, or  
c. Negatively impact long-term maintainability of the code.

The bigger the impact radius, the longer the feedback loop for a team to catch those issues.

![Impact Radius Visualization](https://martinfowler.com/articles/exploring-gen-ai/ai-missteps-categories.png)

#### Impact Radius: Time to Commit

These are the cases where AI hindered me more than it helped. This is actually the least problematic impact radius, because it’s the most obvious failure mode, and the changes most probably will not even make it into a commit.

- **No working code**: At times my intervention was necessary to make the code work, plain and simple. So my experience either came into play because I could quickly correct where it went wrong, or because I knew early when to give up, and either start a new session with AI or work on the problem myself.
  
- **Misdiagnosis of problems**: AI goes down rabbit holes quite frequently when it misdiagnoses a problem. Many of those times I can pull the tool back from the edge of those rabbit holes based on my previous experience with those problems.

  *Example*: It assumed a Docker build issue was due to architecture settings for that Docker build and changed those settings based on that assumption — when in reality, the issue stemmed from copying `node_modules` built for the wrong architecture. As that is a typical problem I have come across many times, I could quickly catch it and redirect.

---

#### Impact Radius: Team Flow in the Iteration

This category is about cases where a lack of review and intervention leads to friction on the team during that delivery iteration.

- **Too much up-front work**: AI often goes broad instead of incrementally implementing working slices of functionality. This risks wasting large upfront work before realizing a technology choice isn’t viable, or a functional requirement was misunderstood.

  *Example*: During a frontend tech stack migration task, it tried converting all UI components at once rather than starting with one component and a vertical slice that integrates with the backend.

- **Brute-force fixes instead of root cause analysis**: AI sometimes took brute-force approaches to solve issues rather than diagnosing what actually caused them.

  *Example*: When encountering a memory error during a Docker build, it increased the memory settings rather than questioning why so much memory was used in the first place.

---

#### Impact Radius: Long-Term Maintainability

This is the most insidious impact radius because it has the longest feedback loop; these issues might only be caught weeks and months later.

- **Verbose and redundant tests**: While AI can be fantastic at generating tests, I frequently find that it creates new test functions instead of adding assertions to existing ones, or that it adds too many assertions, i.e., some that were already covered in other tests.

- **Lack of reuse**: AI-generated code sometimes lacks modularity, making it difficult to apply the same approach elsewhere in the application.

  *Example*: Not realizing that a UI component is already implemented elsewhere, and therefore creating duplicate code.

- **Overly complex or verbose code**: Sometimes AI generates too much code, requiring me to remove unnecessary elements manually.

---

### Conclusions

These experiences mean that by no stretch of my personal imagination will we have AI that writes 90% of our code autonomously in a year. Will it assist in writing 90% of the code? Maybe. For some teams, and some codebases. It assists me in 80% of the cases today (in a moderately complex, relatively small 15K LOC codebase).

![Overview of Main Points](https://martinfowler.com/articles/exploring-gen-ai/ai-missteps-overview.png)

---

### What Can You Do to Safeguard Against AI Missteps?

So how do you safeguard your software and team against the capriciousness of LLM-backed tools, to take advantage of the benefits of AI coding assistants?

#### Individual Coder
- Always carefully review AI-generated code.
- Stop AI coding sessions when you feel overwhelmed.
- Stay cautious of “good enough” solutions that introduce long-term maintenance costs.
- Practice pair programming.

#### Team and Organization
- Good ol’ code quality monitoring.
- Pre-commit hooks and IDE-integrated code review.
- Revisit good code quality practices.
- Make use of custom rules.
- Foster a culture of trust and open communication.
