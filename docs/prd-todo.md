# Product Requirements Document (PRD) - Todo App Upgrade

## 1. Overview

We are upgrading the basic Todo app so users can organize work with due dates, priorities, and date-based filters without adding backend complexity. The goal is a simple, teachable MVP that remains locally stored while making tasks easier to prioritize and review.

---

## 2. MVP Scope

- Add an optional `dueDate` field for each task using ISO `YYYY-MM-DD` format.
- Add a `priority` field with allowed values `P1`, `P2`, and `P3`.
- Default `priority` to `P3` when a new task is created without an explicit value.
- Keep `title` required for all tasks.
- Ignore invalid `dueDate` values and treat them as absent.
- Provide filters for `All`, `Today`, and `Overdue`.
- Show completed tasks in the `All` filter.
- Hide completed tasks in the `Today` and `Overdue` filters.
- Keep all task storage local only, with no backend or external storage changes.

---

## 3. Post-MVP Scope

- Visually highlight overdue tasks so they stand out from non-overdue tasks.
- Add sorting rules in this order: overdue tasks first, then priority from `P1` to `P3`, then due date ascending, with tasks that have no due date listed last.

---

## 4. Out of Scope

- Notifications
- Recurring tasks
- Multi-user support
- Keyboard navigation enhancements
- External or backend-backed storage