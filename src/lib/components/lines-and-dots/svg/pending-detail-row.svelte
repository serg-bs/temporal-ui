<script lang="ts">
  import CodeBlock from '$lib/holocene/code-block.svelte';
  import { translate } from '$lib/i18n/translate';
  import { relativeTime, timeFormat } from '$lib/stores/time-format';
  import type { PendingActivity } from '$lib/types/events';
  import { formatDate } from '$lib/utilities/format-date';
  import {
    formatAttemptsLeft,
    formatRetryExpiration,
  } from '$lib/utilities/format-event-attributes';
  import { formatDuration, getDuration } from '$lib/utilities/format-time';
  import { stringifyWithBigInt } from '$lib/utilities/parse-with-big-int';
  import { toTimeDifference } from '$lib/utilities/to-time-difference';

  import {
    DetailsConfig,
    getPendingEventDetailHeight,
    staticCodeBlockHeight,
  } from '../constants';

  import Box from './box.svelte';
  import Text from './text.svelte';

  export let event: PendingActivity;
  export let x = 0;
  export let y: number;
  export let canvasWidth: number;
  export let active = false;

  const { gutter, fontSizeRatio } = DetailsConfig;

  $: width = canvasWidth / 2;
  $: eventY = y;
  $: textStartingY = eventY + 1.5 * fontSizeRatio;
  $: retrying = event.attempt > 1;
</script>

{#if active}
  <Box
    point={[x, eventY]}
    {width}
    height={getPendingEventDetailHeight(event)}
    fill={retrying ? '#FF9B70' : '#a78bfa'}
  />
{/if}
<Text
  point={[x + gutter, eventY + 0.75 * fontSizeRatio]}
  category="none"
  fontSize="16px"
  fontWeight="500">{retrying ? 'Retrying' : 'Pending'}</Text
>
<foreignObject
  x={x + gutter}
  y={textStartingY}
  width={width - gutter}
  height={fontSizeRatio}
>
  <div class="flex gap-1 text-wrap text-sm text-primary">
    <div class="w-48">{translate('workflows.attempt')}</div>
    {event.attempt}
  </div>
</foreignObject>
<foreignObject
  x={x + gutter}
  y={textStartingY + fontSizeRatio}
  width={width - gutter}
  height={fontSizeRatio}
>
  <div class="flex gap-1 text-wrap text-sm text-primary">
    <div class="w-48">{translate('workflows.attempts-left')}</div>
    {formatAttemptsLeft(event.maximumAttempts, event.attempt)}
  </div>
</foreignObject>
<foreignObject
  x={x + gutter}
  y={textStartingY + 2 * fontSizeRatio}
  width={width - gutter}
  height={fontSizeRatio}
>
  <div class="flex gap-1 text-wrap text-sm text-primary">
    <div class="w-48">{translate('workflows.last-heartbeat')}</div>
    {formatDate(event.lastHeartbeatTime, $timeFormat, {
      relative: $relativeTime,
    }) || '-'}
  </div>
</foreignObject>
<foreignObject
  x={x + gutter}
  y={textStartingY + 3 * fontSizeRatio}
  width={width - gutter}
  height={fontSizeRatio}
>
  <div class="flex gap-1 text-wrap text-sm text-primary">
    <div class="w-48">{translate('workflows.next-retry')}</div>
    {toTimeDifference({
      date: event.scheduledTime,
      negativeDefault: 'None',
    })}
  </div>
</foreignObject>
<foreignObject
  x={x + gutter}
  y={textStartingY + 4 * fontSizeRatio}
  width={width - gutter}
  height={fontSizeRatio}
>
  <div class="flex gap-1 text-wrap text-sm text-primary">
    <div class="w-48">{translate('workflows.expiration')}</div>
    {formatRetryExpiration(
      event.maximumAttempts,
      formatDuration(
        getDuration({
          start: Date.now(),
          end: event.expirationTime,
        }),
      ),
    )}
  </div>
</foreignObject>
{#if event?.heartbeatDetails}
  <foreignObject
    x={x + gutter}
    y={textStartingY + 5 * fontSizeRatio}
    {width}
    height={staticCodeBlockHeight - fontSizeRatio}
  >
    <div class="w-48 text-wrap text-sm text-primary">
      {translate('workflows.heartbeat-details')}
    </div>
    <div class="overflow-auto" style="height: {staticCodeBlockHeight}px">
      <CodeBlock
        content={stringifyWithBigInt(event.heartbeatDetails)}
        copyIconTitle={translate('common.copy-icon-title')}
        copySuccessIconTitle={translate('common.copy-success-icon-title')}
      />
    </div>
  </foreignObject>
{/if}
{#if event?.lastFailure}
  <foreignObject
    x={x + gutter}
    y={event?.heartbeatDetails
      ? textStartingY + 5 * fontSizeRatio + staticCodeBlockHeight
      : textStartingY + 5 * fontSizeRatio}
    {width}
    height={staticCodeBlockHeight - fontSizeRatio}
  >
    <div class="w-48 text-wrap text-sm text-primary">
      {translate('workflows.last-failure')}
    </div>
    <div class="overflow-auto" style="height: {staticCodeBlockHeight}px">
      <CodeBlock
        content={stringifyWithBigInt(event.lastFailure)}
        copyIconTitle={translate('common.copy-icon-title')}
        copySuccessIconTitle={translate('common.copy-success-icon-title')}
      />
    </div>
  </foreignObject>
{/if}
