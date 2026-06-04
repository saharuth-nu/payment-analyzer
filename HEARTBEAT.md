# Heartbeat Tasks

tasks:
- name: daily-summary
  cron: "30 17 * * *"
  prompt: "Generate today's payment summary report from memory log according to AGENTS.md daily summary section"
