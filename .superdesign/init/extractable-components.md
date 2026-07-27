# Extractable components

## AppSidebar

- Source: `src/components/AppSidebar.vue`
- Category: layout
- Description: Persistent study navigation with brand and progress meter.
- Extractable props: `activeTab`, `progress`, `completedCount`, `topicCount`
- Hardcoded: BDA Study Lab brand, navigation labels, icons, CSS classes.

## ThemeWorkspaceNav

- Source: inline in `src/App.vue`
- Category: layout
- Description: Sticky topic header with content, quiz and activities tabs.
- Extractable props: `topicNumber`, `activeSection`
- Hardcoded: tab labels and shared CSS.
