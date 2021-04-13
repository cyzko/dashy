<template>
  <div :class="`collapsable ${checkSpanNum(cols, 'col')} ${checkSpanNum(rows, 'row')}`"
    :style="`${color ? 'background: '+color : ''}; ${sanitizeCustomStyles(customStyles)};`"
  >
    <input
        :id="`collapsible-${uniqueKey}`"
        class="toggle"
        type="checkbox"
        :checked="getCollapseState()"
        @change="collapseChanged"
        tabIndex="-1"
    >
    <label :for="`collapsible-${uniqueKey}`" class="lbl-toggle" tabindex="-1">
      <h3>{{ title }}</h3>
    </label>
    <div class="collapsible-content">
      <div class="content-inner">
        <slot></slot>
        </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CollapsableContainer',
  props: {
    uniqueKey: String,
    title: String,
    collapsed: Boolean,
    cols: Number,
    rows: Number,
    color: String,
    customStyles: String,
  },
  data() {
    return {
      isOpen: !this.collapsed,
    };
  },
  methods: {
    /* Check that row & column span is valid, and not over the max */
    checkSpanNum(span, classPrefix) {
      const maxSpan = 4;
      let numSpan = /^\d*$/.test(span) ? parseInt(span, 10) : 1;
      numSpan = (numSpan > maxSpan) ? maxSpan : numSpan;
      return `${classPrefix}-${numSpan}`;
    },
    /* Removes all special characters, except those allowed in valid CSS */
    sanitizeCustomStyles(userCss) {
      return userCss ? userCss.replace(/[^a-zA-Z0-9- :;.]/g, '') : '';
    },
    /* If not already done, then add object structure to local storage */
    initialiseStorage() {
      const initStorage = () => localStorage.setItem('collapseState', JSON.stringify({}));
      if (!localStorage.collapseState) initStorage(); // If not yet set, then init localstorage
      try { // Check storage is valid JSON, and has not been corrupted
        JSON.parse(localStorage.collapseState);
      } catch {
        initStorage();
      }
      return JSON.parse(localStorage.collapseState);
    },
    getCollapseState() {
      const collapseStateObject = this.initialiseStorage();
      let collapseState = !this.collapsed;
      if (collapseStateObject[this.uniqueKey] !== undefined) {
        collapseState = collapseStateObject[this.uniqueKey];
      }
      return collapseState;
    },
    setCollapseState(id, newState) {
      // Get the current localstorage collapse state object
      const collapseState = JSON.parse(localStorage.collapseState);
      // Add the new state to it
      collapseState[id] = newState;
      // Stringify, and set the new object into local storage
      localStorage.setItem('collapseState', JSON.stringify(collapseState));
    },
    collapseChanged(whatChanged) {
      this.initialiseStorage();
      this.setCollapseState(this.uniqueKey.toString(), whatChanged.srcElement.checked);
    },
  },
};
</script>

<style scoped lang="scss">

@import '@/styles/constants.scss';
@import '@/styles/media-queries.scss';

.collapsable {
    padding: 5px;
    margin: 10px;
    border-radius: $curve-factor;
    background: var(--primary);
    // background: -webkit-linear-gradient(to left top, #9F86FF, #1CA8DD, #007AE1);
    // background: linear-gradient(to left top, #9F86FF, #1CA8DD, #007AE1);
    box-shadow: 1px 1px 2px #130f23;
    height: fit-content;
    width: 100%;
    width: stretch;

    grid-row-start: span 1;
    &.row-2 { grid-row-start: span 2; }
    &.row-3 { grid-row-start: span 3; }
    &.row-4 { grid-row-start: span 4; }

    grid-column-start: span 1;
    @include tablet-up {
      &.col-2 { grid-column-start: span 2; }
      &.col-3 { grid-column-start: span 2; }
      &.col-4 { grid-column-start: span 2; }
    }
    @include laptop-up {
      &.col-2 { grid-column-start: span 2; }
      &.col-3 { grid-column-start: span 3; }
      &.col-4 { grid-column-start: span 3; }
    }
    @include monitor-up {
      &.col-2 { grid-column-start: span 2; }
      &.col-3 { grid-column-start: span 3; }
      &.col-4 { grid-column-start: span 4; }
    }

    .wrap-collabsible {
        margin-bottom: 1.2rem 0;
    }

    input[type='checkbox'] {
        display: none;
    }

    label {
        outline: none;
    }

    .lbl-toggle {
        display: block;
        padding: 0.25rem;
        cursor: pointer;
        border-radius: $curve-factor;
        transition: all 0.25s ease-out;
        text-align: left;
        color: var(--background-transparent);

        h3 {
            margin: 0;
            padding: 0;
            display: inline;
        }
    }

    .lbl-toggle:hover {
        color: var(--background);
    }

    .lbl-toggle::before {
        content: ' ';
        display: inline-block;
        border-top: 5px solid transparent;
        border-bottom: 5px solid transparent;
        border-left: 5px solid currentColor;
        vertical-align: middle;
        margin-right: .7rem;
        transform: translateY(-2px);

        transition: transform .2s ease-out;
    }

    .toggle:checked + .lbl-toggle::before {
        transform: rotate(90deg) translateX(-3px);
    }

    .collapsible-content {
        max-height: 0px;
        overflow: hidden;
        transition: max-height .25s ease-in-out;
        background: var(--background-transparent);
        border-radius: 0 0 $inner-radius $inner-radius;
    }

    .toggle:checked + .lbl-toggle + .collapsible-content {
        max-height: 1000px;
    }

    .toggle:checked + .lbl-toggle {
        border-bottom-right-radius: 0;
        border-bottom-left-radius: 0;
    }

    .collapsible-content .content-inner {
        padding: 0.5rem;
    }
}
</style>