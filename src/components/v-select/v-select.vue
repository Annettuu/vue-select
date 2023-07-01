<template>
  <div
    class="v-select"
  >
    <label
      :uid="uid"
      class="v-select_label"
      @click="toggleList"
    >
      <span class="v-select_title">{{ props.title }}</span>
      <div
        class="v-select_field"
      >
        <input
          ref="input"
          v-model="searchStr"
          class="v-select_input"
          :class="{'__active': isShowList === false && selected.length}"
          @keydown.enter="setFirstKey"
        >
        <div
          v-if="!searchStr"
          class="v-select_chosen"
          :class="{'__highlight': !isToChoose && props.mode === 'multi',
                   '__no-highlight': props.mode !== 'multi',
                   '__translucentText': props.mode === 'single' && !isToChoose
          }"
        >
          {{ stylizedSelected }}
          <button
            v-if="!isToChoose"
            class="v-select_remove"
            @click="resetSelected"
          >
            <img
              src="src/assets/images/svg/remove.svg"
            >
          </button>
        </div>
        <img
          v-if="showCaret"
          class="v-select_caret"
          :class="{'__active': isShowList}"
          src="src/assets/images/svg/caret.svg"
        >
      </div>
    </label>
    <ul
      v-if="isShowList"
      class="v-select_list"
    >
      <div class="v-select_options">
        <li
          v-for="option in autocompleteOptions"
          :key="option.id"
          class="v-select_option"
          :disabled="option.disabled"
          @click="selectOption(option)"
        >
          <v-checkbox
            v-if="Object.hasOwn(option, 'checked') && props.mode === 'multi'"
            :checked="option.checked"
          />
          <span>
            {{ option.content }}
          </span>
        </li>
      </div>
    </ul>
  </div>
</template>

<script setup>
import { ref, defineProps, computed, getCurrentInstance, watchEffect, watch } from 'vue';
import vCheckbox from './v-checkbox.vue';

const props = defineProps({
  title: String,
  options: {
    type: Array
  },
  mode: {
    type: String
  }
});

const uid = getCurrentInstance().uid;
const options = ref(props.mode === 'multi'
  ? props.options.map(o => {
    o.checked = false;

    return o;
  })
  : props.options);
let isShowList = ref(false);
let searchStr = ref('');
let selected = ref('');
const input = ref(null);

watch(props.options, () => {
  let resetTo;
  if (props.mode === 'multi') {
    resetTo = props.options.filter(o => o.checked);
  } else {
    resetTo = props.options.find(o => o.checked)?.content || '';
  }
  selected.value = resetTo;
});

const showCaret = computed(() => {
  let show = false;
  if (props.mode === 'single' && isToChoose.value === false) {
    show = false;

    return show;
  } else if (props.mode !== 'search' ) {
    show = true;
  }

  return show;
});
const autocompleteOptions = computed(() => {
  let filtered;

  if (props.mode === 'multi' || props.mode === 'single') {
    filtered = options.value.filter(option => option.content.toLowerCase().includes(searchStr.value.toLowerCase()));
  } else {
    filtered = props.options.filter(option => option.content.includes(searchStr.value));
  }

  return filtered?.length
    ? filtered
    : [{id: '00', disabled: 'true', content: 'Ничего не найдено'}];
});
const stylizedSelected = computed(() => {
  let stylizedSelected = 'Выбрать';
  if (props.mode === 'multi' && Array.isArray(selected.value)) {
    if (selected.value.length > 1) {
      stylizedSelected = `Выбрано: ${selected.value.length}`;
    }
    if (selected.value.length === 1) {
      stylizedSelected = selected.value.map(v => v.content).join(', ');
    }
  } else {
    selected.value.length && (stylizedSelected = selected.value);
  }

  return stylizedSelected;
});
const isToChoose = computed(() => {
  return stylizedSelected.value.includes('Выбрать');
});

const selectSingle = ({content: value, id: optionId}) => {
  console.debug('single');
  if (selected.value !== value) {
    selected.value = value;

    resetIsShowList();
  } else {
    selected.value = '';
  }
  emit('input', {value: selected.value, mode: props.mode, optionId});
};
const selectMulti = (value) => {
  let result = Array.isArray(selected.value) ? [...selected.value] : [];
  const isExists = !result.some(option => option.id === value.id);
  if (props.mode === 'multi' && isExists) {
    result = [...result, value];
    options.value.find(o => o.id === value.id).checked = true;
  } else {
    const index = result.findIndex(option => option.id === value.id);
    result.splice(index, 1);
    options.value.find(o => o.id === value.id).checked = false;
  }
  result.sort((a,b) => a.id - b.id);
  selected.value = result;
  setInputFocus(true);
  emit('input', {value: selected.value, mode: props.mode, optionId: value.id, checked: value.checked});
};

const selectOption = (option) => {
  if (option.disabled) {
    return;
  }

  if (props.mode === 'multi') {
    selectMulti(option);
  } else {
    selectSingle(option);
  }
};
const setIsShowList = (is) => {
  isShowList.value = is;
};
const setInputFocus = (isFocus) => {
  isFocus ? input.value.focus() : input.value.blur();
};
const resetIsShowList = () => {
  document.removeEventListener('click', handleLabelClick);
  searchStr.value = '';
  setInputFocus(false);
  setIsShowList(false);
};
const resetSelected = () => {
  selected.value = '';
  if (props.mode === 'multi') {
    options.value.forEach(o => o.checked = false);
  }
  setInputFocus(true);
};
const handleLabelClick = (e) => {
  e.stopPropagation();
  e.stopImmediatePropagation();
  e.preventDefault();
  console.debug('handleLabelClick event', e);
  const isOption = e.target.className === 'v-select_option';
  const checkIsOption = !searchStr.value && isOption;
  const isOptionDisabled = e.target.hasAttribute('disabled');
  const isCorrectLabel = e.target.labels && e.target.labels[0]?.className === 'v-select_label' && +e.target.labels[0]?.getAttribute('uid') === uid;
  const showList = checkIsOption || isCorrectLabel;
  if (showList || isOptionDisabled) {
    console.debug('a label');
    if (props.mode !== 'search') {
      setIsShowList(true);
    }
  } else {
    console.debug('not a label');
    resetIsShowList();
  }
};
const toggleList = (e) => {
  if (props.mode === 'single' || props.mode === 'multi') {
    if (isShowList.value && e.target.nodeName === 'INPUT') {
      resetIsShowList();

      return;
    }
    setIsShowList(true);
  }
  document.addEventListener('click', handleLabelClick);
};
const setFirstKey = () => {
  if (autocompleteOptions.value.length && props.mode === 'single') {
    selected.value = autocompleteOptions.value[0].content;
    resetIsShowList();
  }
};
watchEffect(() => {
  if (props.mode === 'search' && autocompleteOptions.value.length > 0 && searchStr.value.length > 0) {
    isShowList.value = true;
  } else {
    isShowList.value = false;
  }
});

</script>

<style lang="scss">
.v-select {
  display: flex;
  align-items: center;
  background-color: transparent;
  flex-direction: column;
  gap: 1px;
  position: relative;

  &_label {
    display: flex;
    flex-direction: column;
    gap: 4px;
  }

  &_title {
    font-weight: 400;
    color: black;
  }

  &_field {
    display: flex;
    align-items: center;
    position: relative;

    &:hover {
      & .v-select_caret {
        filter: brightness(0.5);
      }
    }
  }

  &_input {
    border: 1px solid #66687A;
    border-radius: 6px;
    font-weight: 400;
    color: black;
    padding: 6px 12px 6px 8px;
    width: 280px;

    &:focus-visible {
      outline: none;
      border: 1px solid #3686BB;
    }
    &.__active {
      border: 1px solid #3AA981;
    }
  }

  &_chosen {
    pointer-events: none;
    position: absolute;
    color: #66687A;
    /* Паддинг в инпуте */
    left: 12px;
    top: 50%;
    transform: translateY(-50%);

    &.__highlight {
      display: flex;
      align-items: center;
      border-radius: 8px;
      background-color: #DBE8F1;
      color: #1B1F3B;
      padding-left: 5px;
    }

    &.__no-highlight {
      display: flex;
      justify-content: space-between;
      width: 96.5%;
    }

    &.__translucentText {
      color: #1B1F3B;
    }
  }
  &_remove {
    pointer-events: all;
    cursor: pointer;
    display: flex;
    background: transparent;
    border: none;
  }
  &_caret {
    pointer-events: none;
    position: absolute;
    right: 5px;
    filter: none;
    cursor: pointer;
    &.__active {
      transform: rotate(180deg);
      filter: brightness(0.5);
    }
  }

  &_list {
    display: flex;
    width: 100%;
    max-height: 120px;
    overflow-y: auto;
    flex-direction: column;
    box-shadow: 0px 12px 30px rgba(0, 0, 0, 0.1), 0px 7.7037px 12.7407px rgba(0, 0, 0, 0.0607407), 0px 1.62963px 3.25926px rgba(0, 0, 0, 0.0607);
    position: absolute;
    left: 0;
    top: 100%;
    z-index: 999;
    margin-top: 6px;
    padding: 6px 0 6px 0;
    background: #fff;
    border-radius: 4px;
  }

  &_options {
    overflow-y: auto;
  }

  &_option {
    color: black;
    background-color: white;
    padding: 6px 12px;
    cursor: pointer;

    &:not([disabled="true"]):hover {
      background-color: #EEF5FB;
    }
    & span, & input {
      pointer-events: none;
    }
  }
}

</style>