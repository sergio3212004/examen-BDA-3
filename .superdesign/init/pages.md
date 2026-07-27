# Page dependency trees

## Single-page study application

Entry: `src/App.vue`

- `src/components/AppSidebar.vue`
- `src/components/SummaryView.vue`
  - `src/components/ClusterSimulator.vue`
- `src/components/TopicsView.vue`
  - `src/components/MashupSimulator.vue`
  - `src/components/CbocDeepDive.vue`
- `src/components/PracticeView.vue`
- `src/components/ActivitiesView.vue`
- `src/components/Topic12View.vue`
  - `src/components/Topic12DeepLab.vue`
  - `src/components/FrameworkEncyclopedia.vue`
- `src/components/Topic12Quiz.vue`
- `src/components/Topic12Activities.vue`
- global styles: `src/style.css`

## Proposed Topic 14

The closest representative target is Topic 12:

- `src/App.vue`
- `src/components/AppSidebar.vue`
- `src/components/Topic12View.vue` (structure/style anchor)
- `src/components/Topic12Quiz.vue` (quiz interaction anchor)
- `src/components/Topic12Activities.vue` (activity anchor)
- `src/style.css`
