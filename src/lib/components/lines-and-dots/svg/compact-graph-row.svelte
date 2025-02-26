<script lang="ts">
  import Icon from '$lib/holocene/icon/icon.svelte';
  import type { EventGroup } from '$lib/models/event-groups/event-groups';

  import { CategoryIcon, CompactConfig } from '../constants';

  import Dot from './dot.svelte';
  import Line from './line.svelte';
  import Text from './text.svelte';

  export let group: EventGroup;
  export let count: number = 1;
  export let startIndex: number;
  export let length: number;
  export let y: number;
  export let active = true;
  export let expanded = false;
  export let onClick: () => void;

  const { radius, height } = CompactConfig;

  $: start = 2 * radius + startIndex * length;
  $: end = start + length;
  $: aggregateRow = count > 1;
  $: isPending = group.isPending && !aggregateRow;
</script>

<g
  role="button"
  tabindex="0"
  on:click={onClick}
  on:keypress={onClick}
  class="relative cursor-pointer"
  {height}
>
  <Line
    startPoint={[start, y]}
    endPoint={[end, y]}
    category={group?.pendingActivity
      ? group.pendingActivity.attempt > 1
        ? 'retry'
        : 'pending'
      : group.category}
    classification={aggregateRow ? undefined : group.lastEvent.classification}
    {active}
    strokeWidth={radius * 2}
    pending={isPending}
  />
  <Line
    startPoint={[start, y]}
    endPoint={[end, y]}
    {active}
    status="none"
    strokeWidth={radius}
  />
  <Text point={[start + (4 / 3) * radius, y]} {active} fontWeight="500">
    {#if aggregateRow}<tspan font-weight="700">{count}</tspan>{/if}
    {group?.displayName}
  </Text>
  <Dot
    point={[start, y]}
    classification={aggregateRow ? undefined : group.lastEvent.classification}
    {active}
    r={radius}
  />
  {#if aggregateRow}
    <Icon
      name={CategoryIcon[group.category]}
      x={start - radius / 2}
      y={y - radius / 1.5}
      width={radius}
      height={radius}
      strokeWidth="4"
      class="text-black"
      noDivWrapper
    />
    <Icon
      name={expanded ? 'chevron-up' : 'chevron-down'}
      x={start - radius / 2}
      {y}
      width={radius}
      height={radius}
      strokeWidth="4"
      class="text-black"
      noDivWrapper
    />
  {:else}
    <Icon
      name={CategoryIcon[group.category]}
      x={start - radius / 1.35}
      y={y - radius / 1.35}
      width={radius * 1.5}
      height={radius * 1.5}
      strokeWidth="4"
      class="text-black"
      noDivWrapper
    />
  {/if}
</g>

<style lang="postcss">
  g {
    outline: none;
  }
</style>
