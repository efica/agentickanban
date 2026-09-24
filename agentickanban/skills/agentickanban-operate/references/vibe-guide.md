> Fuente: vibe-guide.html
> Descargado 2026-09-24, convertido de HTML.

Vibe Guide

Vibe Kanban is sunsetting. The project will continue as open source and community maintained.
Read the announcement
# #Planning

I always start with a plan, no matter how small the task is. It doesn't matter whether your agent has a built in 'plan' mode, as you can always ask the agent to "come up with a plan, confirm with me before making changes."

## #Plan more, review less

Five minutes spent planning saves you ten minutes in review. Planning can feel like overhead, but it just moves unavoidable work earlier. A plan is faster to judge than a diff.

## #Planning sets the shape

Agents optimise for getting something working with minimal edits. Once that version exists, subsequent changes patch whatever’s there already, instead of rethinking the approach. The plan is your chance to pick the right shape before the code hardens around the wrong one.

## #Re-plan

If you're making lots of changes to your plan, start a new session. Summarise what you've learned and let the agent begin fresh. This also works when you've made changes, realized they're wrong, and want to try again.

Starting over often beats more feedback. The first attempt shapes everything that follows, so a clean start with better context beats incremental fixes.

At the end of a failed attempt, prompt something like:
Knowing what you know please write a new high level plan:
- No code
- Just sentences
- Mention files to look at, at the bottom of your plan

# #Async

I like working asynchronously with a coding agent (sometimes called backgrounding). It lets me juggle multiple threads at once, and if I don’t, I tend to drift into procrastination. A lot of my workflow is basically: keep the agent running for longer stretches, while I do other things.

## #Use YOLO mode

Async only works if you’re not approving every tool call. Either use your agent’s YOLO mode, or give it an explicit allowlist of commands that covers almost everything it needs. Otherwise you’ve reinvented pair programming, except slower.

Naming will vary but this can be called 'yolo', 'dangerously skip permissions' or similar depending on coding agent.

## #Biggest model is fastest

Use the biggest, most capable, model you can. Smaller models make more mistakes and need more of your time. Same for 'thinking' or 'effort' settings - turn it up to the max. The metric that matters is how often you have to intervene.

With one task, a big model might feel slower. With two or more running in parallel, it's faster. A model that doesn't need your attention frees you to do other things.

## #Set the codebase up to be QA’d

If the agent can't test its own changes, it'll ask you to do it, which is not a great use of your time.

Put in the work to make changes verifiable with a dev or test command. Not everything is testable by an agent (frontend still needs human eyes), but most changes should end with "tests pass," not "can you try this?"

## #Solve dev servers

Agents are bad at starting and cleaning up dev servers, which turns into port conflicts and wasted effort trying to solve "something's already running on that port" errors. I built Dev Manager (https://github.com/BloopAI/dev-manager-mcp) as a small MCP server that assigns ports and kills stale servers to help with this.

## #Add dummy data

Make it easy to run the project offline with realistic seed data. If the agent spends half its time getting the app into a testable state, you'll get flaky results and slower progress.

Dummy data also lets you run multiple agents in parallel. If they share a real database, one might be working on a different schema version than another.

# #Combating laziness

Coding agents make some lazy decisions repeatedly. Here's what I've encountered and how to fix it.

## #No backwards compatibility

Every coding agent I've used defaults to maintaining backwards compatibility. This is part overcaution, part laziness. Keeping the old interface means you don't have to refactor all the call sites.

I add this to my system prompt:

We want the simplest change possible. We don't care about migration. Code readability matters most, and we're happy to make bigger changes to achieve it.

## #Disable disabling lint rules

Agents love to silence linter errors instead of fixing them. eslint-disable-next-line and the like. I've seen this in TypeScript and Rust; it probably happens everywhere.

You can ban this with plugins like eslint-comments/no-restricted-disable.

# #Frontend

My experience with coding agents is limited to React and TypeScript.

## #Separate presentation from logic

I keep leaf components presentational. Business logic (fetching data, etc.) lives in parent components.

Most of my frontend problems with agents involve Frankenstein components: thousands of lines of state management mixed with rendering. Separating concerns makes it easy to audit a presentational component's props and spot irregularities.

It also helps the agent. Agents look for patterns to follow. If you have separate folders for separate concerns, you give them clear examples.

This is semi-enforceable via ESLint by disabling hooks like useState or useEffect in presentational components.
'no-restricted-syntax': [
 'error',
 {
 selector: 'CallExpression[callee.name="useState"]',
 message: 'View components should not manage state. Use controlled props.',
 },
]

## #Restrict Tailwind

Agents love adding custom colors, spacing, and padding everywhere. We use ESLint to enforce a small set of allowed utility classes. For example, p-4 and p-8 are banned, but p-base and p-double are allowed and defined in our Tailwind config.

## #Figma MCP

The Figma MCP server is good for a first pass at presentational components. Select a component in Figma, then prompt:

- The component has been designed in Figma and can be retrieved using the Figma MCP server

- All spacing, colors, and sizes should match existing Tailwind variables (see frontend/tailwind.config.js and frontend/src/styles/index.css)

- We use Phosphor icons

- If you cannot match an existing variable, ask the user what to do
