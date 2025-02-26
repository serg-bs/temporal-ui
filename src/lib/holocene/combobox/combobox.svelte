<script lang="ts">
  import type { HTMLInputAttributes } from 'svelte/elements';
  import { writable } from 'svelte/store';

  import { createEventDispatcher } from 'svelte';
  import { twMerge as merge } from 'tailwind-merge';

  import ComboboxOption from '$lib/holocene/combobox/combobox-option.svelte';
  import MenuContainer from '$lib/holocene/menu/menu-container.svelte';
  import Menu from '$lib/holocene/menu/menu.svelte';

  import type { IconName } from '../icon';
  import Icon from '../icon/icon.svelte';

  type T = $$Generic;

  const dispatch = createEventDispatcher<{
    change: { value: string | T };
    filter: string;
    close: { selectedOption: string | T };
  }>();

  type ExtendedInputEvent = Event & {
    currentTarget: EventTarget & HTMLInputElement;
  };

  interface BaseProps extends HTMLInputAttributes {
    id: string;
    label: string;
    toggleLabel: string;
    value: string;
    noResultsText: string;
    disabled?: boolean;
    labelHidden?: boolean;
    placeholder?: string;
    readonly?: boolean;
    required?: boolean;
    leadingIcon?: IconName;
    minSize?: number;
    maxSize?: number;
    'data-testid'?: string;
    error?: string;
    valid?: boolean;
  }

  type UncontrolledStringOptionProps = {
    options: string[];
    optionValueKey?: never;
    optionLabelKey?: never;
    displayValue?: never;
  };

  type UncontrolledCustomOptionProps = {
    options: T[];
    optionValueKey: keyof T;
    optionLabelKey?: keyof T;
  };

  type $$Props =
    | (BaseProps & UncontrolledStringOptionProps)
    | (BaseProps & UncontrolledCustomOptionProps);

  let className = '';
  export { className as class };
  export let id: string;
  export let label: string;
  export let value: string = undefined;
  export let toggleLabel: string;
  export let noResultsText: string;
  export let disabled = false;
  export let labelHidden = false;
  export let options: (T | string)[];
  export let placeholder: string = null;
  export let readonly = false;
  export let required = false;
  export let leadingIcon: IconName = null;
  export let optionValueKey: keyof T = null;
  export let optionLabelKey: keyof T = optionValueKey;
  export let minSize = 0;
  export let maxSize = 120;
  export let error = '';
  export let valid = true;

  let displayValue: string;
  let selectedOption: string | T;
  let menuElement: HTMLUListElement;
  let inputElement: HTMLInputElement;
  const open = writable<boolean>(false);
  $: list = options;

  $: {
    if (inputElement && displayValue) {
      if (displayValue.length < minSize) {
        inputElement.size = minSize;
      } else if (displayValue.length > maxSize) {
        inputElement.size = maxSize;
      } else {
        inputElement.size = displayValue.length;
      }
    }
  }

  $: {
    selectedOption = options.find((option) => {
      if (isStringOption(option)) {
        return option === value;
      }

      if (isObjectOption(option) && canRenderCustomOption(option)) {
        return option[optionValueKey] === value;
      }
    });

    displayValue = getDisplayValue(selectedOption);
  }

  const toggleList = () => {
    if ($open) {
      closeList();
    } else {
      openList();
    }
  };

  const openList = () => {
    $open = true;
    inputElement.focus();
    inputElement.select();
  };

  const closeList = () => {
    if (!$open) return;
    $open = false;
    dispatch('close', { selectedOption });
    resetValueAndOptions();
  };

  const handleMenuClose = () => {
    dispatch('close', { selectedOption });
    resetValueAndOptions();
  };

  const resetValueAndOptions = () => {
    displayValue = getDisplayValue(selectedOption);
    list = options;
  };

  const narrowOption = (option: unknown): T => option as T;

  const isStringOption = (option: string | T): option is string => {
    return typeof option === 'string';
  };

  const isObjectOption = (option: string | T): option is T & object => {
    return typeof option === 'object';
  };

  const canRenderCustomOption = (option: T) => {
    if (option === null) return false;
    if (typeof option !== 'object') return false;

    return (
      optionValueKey !== null &&
      optionLabelKey !== null &&
      optionValueKey in option &&
      optionLabelKey in option
    );
  };

  const getDisplayValue = (option: string | T | undefined): string => {
    if (!option) return value ?? '';

    if (isStringOption(option)) {
      return option;
    }

    if (isObjectOption(option) && canRenderCustomOption(option)) {
      return String(option[optionLabelKey]);
    }
  };

  const setValue = (option: string | T): void => {
    if (isStringOption(option)) {
      value = option;
    }

    if (isObjectOption(option) && canRenderCustomOption(option)) {
      value = String(option[optionValueKey]);
    }
  };

  const handleSelectOption = (option: string | T) => {
    setValue(option);
    dispatch('change', { value: option });
    resetValueAndOptions();
  };

  const focusFirstOption = () => {
    const listItemElement: HTMLLIElement = menuElement.querySelector(
      'li[role="option"]:not([aria-disabled="true"])',
    );

    if (listItemElement) {
      requestAnimationFrame(() => listItemElement.focus());
    }
  };

  const handleInputKeydown = (event: KeyboardEvent) => {
    switch (event.key) {
      case 'Escape':
        closeList();
        break;
      case 'ArrowDown':
        openList();
        focusFirstOption();
        break;
      case 'Enter':
        openList();
        focusFirstOption();
        break;
      case 'ArrowUp':
      case 'ArrowRight':
      case 'ArrowLeft':
        break;
      default:
        break;
    }
  };

  const handleInput = (event: ExtendedInputEvent) => {
    displayValue = event.currentTarget.value;
    dispatch('filter', displayValue);
    if (!$open) $open = true;

    list = options.filter((option) => {
      if (isStringOption(option)) {
        return option
          .toLowerCase()
          .includes(event.currentTarget.value.toLowerCase());
      }

      if (isObjectOption(option) && canRenderCustomOption(option)) {
        return String(option[optionLabelKey])
          .toLowerCase()
          .includes(event.currentTarget.value.toLowerCase());
      }
    });
  };

  const handleInputClick = () => {
    if (!$open) openList();
  };
</script>

<MenuContainer {open} on:close={handleMenuClose}>
  <label
    class="combobox-label"
    class:sr-only={labelHidden}
    class:required
    for={id}
  >
    {label}
  </label>
  <div class="combobox-wrapper" class:disabled class:invalid={!valid}>
    {#if leadingIcon}
      <Icon
        width={20}
        height={20}
        class="ml-2 shrink-0"
        name={leadingIcon}
        noDivWrapper
      />
    {/if}
    <input
      {id}
      {placeholder}
      {required}
      {readonly}
      {disabled}
      type="text"
      value={displayValue}
      class:disabled
      class={merge('combobox-input', className)}
      role="combobox"
      autocomplete="off"
      autocapitalize="off"
      spellcheck="false"
      data-lpignore="true"
      aria-controls="{id}-listbox"
      aria-expanded={$open}
      aria-required={required}
      aria-autocomplete="list"
      on:focus|stopPropagation={openList}
      on:input|stopPropagation={handleInput}
      on:keydown|stopPropagation={handleInputKeydown}
      on:click|stopPropagation={handleInputClick}
      data-testid={$$props['data-testid'] ?? id}
      bind:this={inputElement}
      {...$$restProps}
    />
    <button
      aria-label={toggleLabel}
      class="combobox-button"
      tabindex={-1}
      aria-controls="{id}-listbox"
      aria-expanded={$open}
      type="button"
      on:click={toggleList}
    >
      <Icon name={$open ? 'chevron-up' : 'chevron-down'} />
    </button>
  </div>
  {#if error && !valid}
    <span class="error">{error}</span>
  {/if}

  <Menu bind:menuElement id="{id}-listbox" role="listbox" class="w-full">
    {#each list as option}
      {#if isStringOption(option)}
        <ComboboxOption
          on:click={() => handleSelectOption(option)}
          selected={value === option}
        >
          {option}
        </ComboboxOption>
      {:else if isObjectOption(option)}
        {#if canRenderCustomOption(option)}
          <ComboboxOption
            on:click={() => handleSelectOption(option)}
            selected={value === option[optionValueKey]}
          >
            {option[optionLabelKey]}
          </ComboboxOption>
        {:else}
          <slot option={narrowOption(option)} />
        {/if}
      {/if}
    {:else}
      <ComboboxOption disabled>{noResultsText}</ComboboxOption>
    {/each}
  </Menu>
</MenuContainer>

<style lang="postcss">
  .combobox-label {
    @apply font-secondary text-sm font-medium text-primary;

    &.required {
      @apply after:content-['*'];
    }
  }

  .combobox-wrapper {
    @apply surface-primary flex h-10 w-full flex-row items-center rounded-lg border border-primary text-sm dark:focus-within:surface-primary focus-within:border-interactive focus-within:shadow-focus focus-within:shadow-primary/50 focus-within:outline-none dark:bg-transparent;

    &.invalid {
      @apply border-2 border-error text-danger focus-within:shadow-danger/50;
    }

    &.disabled {
      @apply surface-disabled border-subtle text-disabled;
    }
  }

  .error {
    @apply text-xs text-danger;
  }

  .combobox-input {
    @apply ml-2 h-full w-full grow bg-transparent font-primary text-primary placeholder:text-primary focus:outline-none disabled:text-disabled disabled:placeholder:text-disabled dark:bg-transparent;
  }

  .combobox-button {
    @apply mx-2 flex shrink-0 items-center justify-center rounded-full hover:surface-interactive-secondary;
  }
</style>
