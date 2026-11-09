# Course Engine skill for Claude and other agents

An agent skill for [Course Engine](https://course-engine.com), the platform for schools that
sell online courses in six languages. It teaches your agent to work in your school through the
Course Engine MCP server: building courses with AI step by step with cost previews, students and
orders, enrolments and certificates, coupons and promotions, reviews and forums. It also covers
the optional Unison, Support CX and Meta ads integrations. It always follows the
preview-and-confirm rules for anything customer-facing.

## Install

```bash
npx skills add SolutionLabsIO/course-engine-skills
```

Add `-g` to install it for all your projects, or `-a claude-code` to pick the agent. The
[skills CLI](https://github.com/vercel-labs/skills) supports Claude Code, Cursor, Codex and many
other agents.

## Connect your school

The skill needs the Course Engine MCP server. Connect it with your own school admin account:

- **Server URL:** `https://mcp.course-engine.com/mcp`
- **claude.ai / Claude Desktop:** Settings → Connectors → Add custom connector → paste the URL →
  Connect.
- **Claude Code:** `claude mcp add --transport http course-engine https://mcp.course-engine.com/mcp`,
  then `/mcp` to sign in.

You sign in to the Course Engine admin with Google, or a password plus two-factor, and click
Allow. Claude then works as you, with your permissions, in your school only. Every change is
recorded in your school's audit log under your name.

## What is in the repo

```
skills/
  course-engine/
    SKILL.md    the skill: connection, workflows, tool names, safety rules
```

## Safety

- Paid access, coupons, publishing, customer replies, scheduled broadcasts and progress resets
  first return a preview and a one-time code. Nothing happens until you say yes.
- The connector never moves money: refunds, prices, subscriptions and ad budgets stay in the web
  admin.
- AI work shows its estimated cost and your school's budget before it runs.
- No tool deletes a course, a student, an order or a ticket; the few actions that cannot be
  undone are previewed and confirmed first.

More: https://course-engine.com/connectors/claude

## License

MIT. See [LICENSE](LICENSE).
