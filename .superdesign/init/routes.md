# Route map

This is a Vite + Vue 3 single-page application without Vue Router. `src/App.vue` controls view navigation through local state.

| Virtual route | State | Component/layout |
|---|---|---|
| Summary | `tab = resumen` | `SummaryView` inside sidebar shell |
| Topic 11 | `tab = temas` | `TopicsView`, `PracticeView`, `ActivitiesView` |
| Topic 12 content | `tab = tema12`, `theme12Section = contenido` | `Topic12View` inside theme workspace |
| Topic 12 quiz | `tab = tema12`, `theme12Section = quiz` | `Topic12Quiz` |
| Topic 12 activities | `tab = tema12`, `theme12Section = actividades` | `Topic12Activities` |
| Proposed Topic 14 | `tab = tema14` | New content, quiz and activities views using the same workspace pattern |

Entry flow: `index.html` → `src/main.ts` → `src/App.vue`.
