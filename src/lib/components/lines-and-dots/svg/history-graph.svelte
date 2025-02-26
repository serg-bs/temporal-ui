<script lang="ts">
  import { scrollTop } from '$lib/holocene/main-content-container.svelte';
  import { groupWorkflowTaskEvents } from '$lib/models/event-groups';
  import type { EventGroups } from '$lib/models/event-groups/event-groups';
  import {
    endIndex,
    indexPageSize,
    startIndex,
  } from '$lib/stores/active-events';
  import type { WorkflowEvents } from '$lib/types/events';

  import { getEventDetailsBoxHeight, HistoryConfig } from '../constants';

  import EventDetailsRow from './event-details-row.svelte';
  import HistoryGraphRowVisual from './history-graph-row-visual.svelte';
  import HistoryGraphRow from './history-graph-row.svelte';
  import Line from './line.svelte';

  export let history: WorkflowEvents;
  export let groups: EventGroups;
  export let activeEvents: string[] = [];
  export let canvasWidth: number;
  export let zoomLevel: number = 1;

  $: workflowTaskGroups = groupWorkflowTaskEvents(history);
  $: allGroups = [...workflowTaskGroups, ...groups];

  const { height } = HistoryConfig;

  const setHistorySlice = (top: number) => {
    let scrollIndex = Math.round(top / height);

    if ($endIndex - scrollIndex < indexPageSize / 4) {
      $endIndex += indexPageSize;
    } else if ($startIndex > scrollIndex && scrollIndex < indexPageSize / 4) {
      $startIndex = 0;
    }
  };

  $: setHistorySlice($scrollTop);

  $: visibleHistory = history.slice($startIndex, $endIndex);

  $: activeDetailsHeight = activeEvents
    .map((id) => {
      const event = history.find((event) => event.id === id);
      const group = allGroups.find((group) => group.eventIds.has(id));
      if (group) {
        return group.eventList.reduce(
          (sum, event) =>
            (sum += getEventDetailsBoxHeight(event, group.pendingActivity)),
          0,
        );
      }
      return getEventDetailsBoxHeight(event);
    })
    .reduce((acc, height) => acc + height, 0);

  $: canvasHeight = Math.max(
    visibleHistory.length * height + activeDetailsHeight + height,
    400,
  );
  $: visualWidth = canvasWidth / 4;
</script>

<svg
  viewBox="0 0 {canvasWidth} {canvasHeight}"
  height={canvasHeight}
  width={canvasWidth}
>
  <Line
    startPoint={[visualWidth, 0]}
    endPoint={[visualWidth, canvasHeight]}
    strokeWidth={4}
  />
  {#each visibleHistory as event, index (event.id)}
    <HistoryGraphRow
      {event}
      group={allGroups.find((g) => g.eventIds.has(event.id))}
      {activeEvents}
      {canvasWidth}
      {index}
    />
  {/each}
  {#each activeEvents as id}
    <EventDetailsRow
      x={canvasWidth / 2 + 2}
      event={history.find((event) => event.id === id)}
      group={allGroups.find((group) => group.eventIds.has(id))}
      index={visibleHistory.indexOf(history.find((event) => event.id === id))}
      {canvasWidth}
    />
  {/each}
  <svg
    viewBox="0 0 {2 * canvasWidth} {canvasHeight * zoomLevel}"
    height={canvasHeight}
    width={(2 * canvasWidth) / zoomLevel}
  >
    {#each visibleHistory as event, index (event.id)}
      <HistoryGraphRowVisual
        {event}
        group={allGroups.find((g) => g.eventIds.has(event.id))}
        groups={allGroups}
        {history}
        {canvasWidth}
        {activeEvents}
        {zoomLevel}
        {index}
      />
    {/each}
  </svg>
</svg>
