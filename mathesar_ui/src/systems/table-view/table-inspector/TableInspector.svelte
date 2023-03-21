<script lang="ts">
  import { TabContainer } from '@mathesar/component-library';
  import { getTabularDataStoreFromContext } from '@mathesar/stores/table-data';
  import type { ComponentType } from 'svelte';
  import ColumnMode from './column/ColumnMode.svelte';
  import RecordMode from './record/RecordMode.svelte';

  import TableMode from './table/TableMode.svelte';

  type TabItem = { label: string; id: number; component: ComponentType };
  const tabs: TabItem[] = [
    {
      label: 'Table',
      component: TableMode,
      id: 1,
    },
    {
      label: 'Column',
      component: ColumnMode,
      id: 2,
    },
    {
      label: 'Record',
      component: RecordMode,
      id: 3,
    },
  ];

  let activeTab: TabItem;

  const tabularData = getTabularDataStoreFromContext();
  $: ({ selection } = $tabularData);
  $: ({ selectedCells } = selection);

  $: {
    // Explicit dependency
    $selectedCells;

    if (selection.isAnyColumnCompletelySelected()) {
      [, activeTab] = tabs;
    }

    if (selection.isAnyRowCompletelySelected()) {
      [, , activeTab] = tabs;
    }
  }


  let isResizing = false;
  let startingPointerX: number | undefined;
  let startingColumnWidth: number | undefined;
  let minInspectorwidth: 40;

  function isTouchEvent(e: MouseEvent | TouchEvent): e is TouchEvent {
    return 'touches' in e;
  }

  function getPointerX(e: MouseEvent | TouchEvent) {
    const singularEvent = isTouchEvent(e) ? e.touches[0] : e;
    return singularEvent.clientX;
  }

  function resize(e: MouseEvent | TouchEvent) {
    const pointerMovement = getPointerX(e) - (startingPointerX ?? 0);
    const newColumnWidth = Math.max(
      (startingColumnWidth ?? 0) + pointerMovement,
      minInspectorwidth,
    );
    // Update the width of the sidebar
    const container = document.querySelector('.table-inspector-container') as HTMLElement;
    container.style.width = `${newColumnWidth}px`;
  }

  function stopColumnResize() {
    isResizing = false;
    startingPointerX = undefined;
    startingColumnWidth = undefined;
    window.removeEventListener('mousemove', resize, true);
    window.removeEventListener('touchmove', resize, true);
    window.removeEventListener('mouseup', stopColumnResize, true);
    window.removeEventListener('touchend', stopColumnResize, true);
    window.removeEventListener('touchcancel', stopColumnResize, true);
    // Remove the class that indicates that the user is resizing the sidebar
    const resizer = document.querySelector('.inspector-resizer') as HTMLElement;
    resizer.classList.remove('is-resizing');
  }

  function startColumnResize(e: MouseEvent | TouchEvent) {
    isResizing = true;
    startingColumnWidth = undefined;
    startingPointerX = getPointerX(e);
    window.addEventListener('mousemove', resize, true);
    window.addEventListener('touchmove', resize, true);
    window.addEventListener('mouseup', stopColumnResize, true);
    window.addEventListener('touchend', stopColumnResize, true);
    window.addEventListener('touchcancel', stopColumnResize, true);
    // Add the class that indicates that the user is resizing the sidebar
    const resizer = document.querySelector('.inspector-resizer') as HTMLElement;
    resizer.classList.add('is-resizing');
  }
  
</script>

<div class="table-inspector-container">
  <div
    class="inspector-resizer" 
    class:is-resizing={isResizing}
    on:dragstart
    on:mousedown={startColumnResize}
    on:touchstart|nonpassive={startColumnResize}
  >
    <TabContainer
      bind:activeTab
      {tabs}
      tabStyle="compact"
      fillContainerHeight
      fillTabWidth
    >
      <slot>
        {#if activeTab}
          <div class="tabs-container">
            <svelte:component this={activeTab.component} />
          </div>
        {/if}
      </slot>
    </TabContainer>
    </div>
</div>


<style lang="scss">
  .table-inspector-container {
    width: var(--table-inspector-width, 400px);
    box-shadow: 0px 2px 2px 0px rgba(0, 0, 0, 0.14),
      0px 3px 1px -2px rgba(0, 0, 0, 0.12), 0px 1px 5px 0px rgba(0, 0, 0, 0.2);
    position: relative;
    background-color: var(--sand-100);
    isolation: isolate;

    :global(.collapsible > .collapsible-header > button.btn) {
      background-color: var(--sand-200);

      &:hover {
        background-color: var(--sand-300);
      }

      &:active {
        background-color: var(--sand-400);
      }
    }
  }
  .tabs-container{
    width: 400px;
    position: relative;
    height: 100%;
  }
  .inspector-resizer {
    --width: 400px;
    position: absolute;
    width: var(--width);
    right: calc(-1 * var(--width) / 2);
    z-index: var(--z-index__sheet__column-resizer);
    cursor: e-resize;
    display: flex;
    justify-content: center;
  }
</style>
