---
name: course-engine
description: Run an online school on Course Engine from Claude through its MCP server. Use when someone wants to build or review a course with AI, check AI spend, look up students and orders, manage enrolments, certificates, coupons, promotions, reviews and forums, or (where the school has them) Unison marketing, Support CX tickets and draft Meta ads. Covers connecting, the tool to use for each job, and the preview-and-confirm safety rules.
---

# Course Engine

Course Engine is a platform for schools that sell online courses. One school can sell in six
languages (English, French, German, Spanish, Italian, Dutch), with one course per language.
It handles course creation with AI, lessons, quizzes and final exams, certificates, the shop,
checkout, subscriptions, coupons and promotions, forums and reviews.

Its MCP server lets a school admin do this work from Claude, signed in as themselves.

## Connect first

The tools below exist only once the Course Engine connector is connected. If you cannot see
tools named `whoami`, `ce_*` or `student_360`, tell the person how to connect and stop there.

- **Server URL:** `https://mcp.course-engine.com/mcp`
- **claude.ai / Claude Desktop:** Settings → Connectors → Add custom connector → paste the
  URL → Connect.
- **Claude Code:** `claude mcp add --transport http course-engine https://mcp.course-engine.com/mcp`,
  then run `/mcp` to sign in.

Sign-in is OAuth. The person signs in to the Course Engine admin with **Google, or a password
plus two-factor** (a password alone is refused), then clicks Allow. Only school admins can
connect.

Claude then acts **as that admin, with that admin's permissions, inside that admin's school
only**. Roles are re-checked on every call, so removing someone's admin role cuts their access
at once. Every change goes into the school's audit log under the person's name.

Call `whoami` at the start to confirm who is signed in and which school it is.

In some clients the tool names carry a prefix (for example `mcp__course-engine__ce_get_order`).
This guide uses the bare names.

## Safety rules (always)

1. **Preview, then confirm.** A tool that reaches customers or changes what they see (paid
   access, coupons, publishing, customer replies, scheduling broadcasts, resetting progress)
   first answers with a **PREVIEW and a one-time confirmation code**. Nothing has happened yet.
   This connector never moves money: refunds, price changes, subscription changes and ad
   budgets are done in the Course Engine web admin.
2. **Show the preview and wait for a clear yes.** Show the person what the preview says:
   recipients, amounts, texts, dates. Only after they say yes, call the same tool again with
   **exactly the same arguments** plus `confirmation_code`.
3. **Never invent a code** and never reuse one from another preview. Only use a code the tool
   returned in this conversation and the person approved. Codes expire after about 10 minutes
   and work once. If one has expired, call again without a code to get a new preview.
4. **AI steps show their cost first.** Anything that runs AI costs the school money. Say the
   estimate and the school's remaining budget, and go ahead only after a yes. Never raise a
   cost cap beyond what the person approved. If the cost is unknown, say "unknown" (never
   "free" or "$0") and continue only if the person accepts that.
5. **When Course Engine refuses because something changed** since you read it, say so, read
   it again, and ask before retrying.
6. **No tool deletes a course, a student, an order or a ticket.** The few actions that cannot be
   undone (resetting a student's progress, merging two support tickets) always show a preview
   and need the person's confirmation code. Otherwise offer to close, archive, hide or unpublish.
7. Drafts are safe. Approving course drafts, writing a draft ad or a draft course changes
   nothing students or customers see. Say so when it applies.

## Build a course with AI, step by step

A course project moves through phases (outline, lessons, pictures, quizzes, final review).
Projects are started in the web admin; from Claude you follow them, review them and move them
forward.

1. `ce_course_status`: what is waiting in each project, and on whom (the school, or Course
   Engine while the AI works). Start here.
2. `ce_course_outline`: modules and lessons with durations, objectives and, once written,
   each lesson's `content_id` and review status.
3. `ce_course_topic`: read one written lesson as text, with its open comments.
4. `ce_course_questions` → `ce_course_answer`: settle the points the sources did not settle,
   in the expert's own words. Answers can be changed later.
5. `ce_course_comment` → `ce_course_fix`: leave a comment on a lesson, then `propose` a fix
   (shows before/after), `accept` it (applies to the draft) or `revert` it.
6. `ce_course_approve`: approve one lesson, a whole layer, or all generated pictures. This
   publishes nothing to students.
7. `ce_course_advance`: move to the next step. The first call is a **cost preview** (the
   step, the estimated AI cost, the school's spend and budget). Tell the person the cost.
   After their yes, call again with `max_cost_cents` set to the previewed estimate, plus the
   confirmation code when the preview gives one.
8. `ce_course_resume`: restart AI work that the school's spending controls paused (previewed).
9. `ce_ai_spend`: AI spending this period, the plan allowance and the monthly budget, at any
   time. The budget itself is changed only in the web admin.

Also:

- **"Awaiting the instructor" notes** are placeholders the AI left where a lesson needs an
  answer. A lesson cannot be approved, and the review cannot be left, while they show. Get the
  question answered, then `ce_course_answer` with action `resolve_notes`, and accept or reject
  each proposed edit with `ce_course_fix`.
- `ce_course_quiz_counts`: question counts per lesson quiz and for the final exam. Lowering a
  target keeps the best questions and writes none; show the list of questions that would be
  removed before confirming.
- `ce_school_rules`: the school's own writing rules, followed for every course. Adding or
  accepting a rule changes all future AI writing: show the wording and wait for a yes.

### Edit courses by hand

`ce_catalog_courses`, `ce_catalog_topic`, `ce_catalog_create_course` (a draft),
`ce_catalog_update_course`, `ce_catalog_add_lesson`, `ce_catalog_add_topic`,
`ce_catalog_update_topic`, `ce_catalog_set_course_status` (publish or back to draft).
Changes to a published course are previewed first.

## Students and orders

- `student_360`: everything about one person by email (profile, enrolments, orders,
  subscriptions, certificates, progress, plus marketing and support history where the school
  has those). Start here for any question about one person.
- Find and read: `ce_find_students`, `ce_get_student`, `ce_list_orders`, `ce_get_order`,
  `ce_list_subscriptions`, `ce_email_log`, `ce_catalog_enrolments`.
- Act:
  - `ce_add_order_note` (internal, no confirmation);
  - `ce_grant_access` / `ce_revoke_access`, `ce_catalog_enrol` / `ce_catalog_unenrol`;
  - `ce_reset_course_progress` (cannot be undone, confirmed first), `ce_certificate_action`;
  - `ce_send_account_email` (activation or sign-in email), `ce_resend_logged_email`,
    `ce_sign_out_everywhere`.
- Refunds and subscription changes are not available here: send the person to the web admin
  (Commerce → Orders or Subscriptions).
- Overview: `ce_dashboard`, `ce_online_now`, `ce_catalog_cpd_completions`.

## Products, coupons and promotions

- Products: `ce_catalog_products`, `ce_get_product`, `ce_catalog_create_product` (a draft, no
  price). Prices and pricing plans are set in the web admin (Commerce → Products).
- Coupons: `ce_list_coupons`, `ce_create_coupon`, `ce_disable_coupon`,
  `ce_close_coupon_campaign`.
- Site banner and course-page banner texts: `ce_list_promotions`,
  `ce_update_promotion_banner`. It can translate the other languages in the same save. A saved
  change shows on the site within a few minutes.
- Gifts: `ce_list_gifts` (read only).
- A/B tests on sales pages: `ce_list_experiments`, `ce_experiment_action`.

## Reviews, forums and quality queues

- Reviews: `ce_list_reviews`, `ce_reply_to_review` (public, confirmed),
  `ce_set_review_status` (publish or hide).
- Forums: `ce_forum_topics`, `ce_forum_topic`, `ce_moderate_forum` (sticky, close, move,
  hide).
- `ce_instructor_note_action`: approve or reject instructors' notes on lessons.

Slide decks, narrated videos, AI pictures and audio are made in the web admin, not through
this connector. Course steps that generate pictures are refused here too
(`ce_course_advance` says so).

## Optional integrations

Only some schools have these. If a tool answers that the school is not connected, say so and
move on.

- **Unison** (email, SMS and social marketing; `unison_*`): contacts, tags, automations,
  templates (with AI translation into the six languages), broadcasts and social posts.
  Broadcasts and posts are **only scheduled, at least 15 minutes ahead, after a preflight
  check**; there is no send-now. Test sends go to the person's own address. Useful entry
  points: `unison_find_contact`, `unison_list_automations`, `unison_create_broadcast`,
  `unison_broadcast_preflight`, `unison_schedule_broadcast`, `unison_stop_sending`.
- **Support CX** (help desk; `support_*`): search and read tickets and chats, AI drafts
  (nothing sent), internal notes, status and tags, and replies to customers (previewed,
  confirmed). Tickets are archived, never deleted.
- **Meta ads** (through Unison; `unison_*ad*`, `ads_*`):
  - New campaigns and ads are drafts with nothing in Meta.
  - `unison_build_ad_campaign` creates everything **paused**.
  - Switching campaigns or ads on and changing budgets are not available here: that is done
    in the web admin, so nothing this connector drafts can start spending.
  - `ads_performance` and `ads_recommendations` compare Meta's numbers with the school's own
    orders. Recommendations are only suggestions.
  - A file attached to the chat cannot be passed to a tool. For a pack of ad pictures, create
    an upload link with `unison_create_upload_link`, let the person upload in their browser,
    then read it with `unison_list_uploaded_files`.

## Good habits

- Read before you write: fetch the order, student or course first, and quote what you
  will change.
- Name money in the currency the tool gives. AI costs are billed in the currency the preview
  shows; ad budgets are in the ad account's currency.
- Ask one clear question when a request is ambiguous (which course, which language, which
  payment) rather than guessing.
- After a change, read the object again and tell the person the result in one or two lines.
