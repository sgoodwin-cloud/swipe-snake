# Swipe Snake — notes for Claude

Read `DESIGN.md` first. It holds the game spec, code structure, release process, milestones and decision log.

- Keep the game fully offline: no runtime network calls.
- On every release, bump `APP_VERSION` (and the `ver` span) in `index.html` and `VERSION` in `sw.js` together.
- Add any new file to `CORE` in `sw.js`.
- Update `DESIGN.md` (milestone ticks, decision log, current-state section, "Last updated" line) in the same commit as the change.
- UK spelling throughout.
