<template>
  <div class="floatlabel-input" v-click-outside="hide">
    <div class="floatlabel-input__content">
      <input
        class="floatlabel-input__input"
        :type="type"
        min="0"
        @blur="removeLabelStyles($event.target)"
        @focus="handleLabelStyles($event.target), showSuggestions = true"
        @input="filterSuggestions($event.target.value)"
        v-model="inputValue"
        :title="title"
        :disabled="disabled"
        :pattern="pattern"
      />
      <div class="floatlabel-input__label" ref="label-container">
        <p class="floatlabel-input__text-label" htmlFor="">{{
          label
        }}</p>
      </div>
    </div>
    <div class="suggestions" v-if="suggestions.length && showSuggestions">
      <ul class="suggestions-list">
        <li class="suggestions-list__item" @click="selectSuggestion(suggestion)" v-for="(suggestion, index) in filteredSuggestions" :key="index">
          {{ suggestion }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: "FloatInput",
  props: {
    label: {
      type: String,
      required: false,
    },
    type: String,
    disabled: {
      default: false,
      type: Boolean,
      required: false,
    },
    pattern: {
      type: String,
      required: false,
    },
    suggestions: {
      type: Array,
      required: false,
      default: () => [],
    },
  },
    directives: {
    "click-outside": {
      bind: (el, binding, vnode) => {
        el.clickOutsideEvent = (event) => {
          if (!(el === event.target || el.contains(event.target))) {
            vnode.context[binding.expression](event);
          }
        };
        document.body.addEventListener("click", el.clickOutsideEvent);
      },
      unbind: (el) => {
        document.body.removeEventListener("click", el.clickOutsideEvent);
      },
    },
  },
  data() {
    return {
      inputValue: "",
      showSuggestions: false,
      filteredSuggestions: []
    };
  },
  watch: {
    inputValue(value) {
      this.$emit("input", value);
    },
    $attrs(){
      this.inputValue = this.$attrs.value;
    }
  },
  computed: {
    title() {
      if (this.inputValue) {
        return `${this.label}: ${this.inputValue}`;
      }
      return this.label;
    },
  },
  created() {
    this.inputValue = this.$attrs.value;
    this.filteredSuggestions = [...this.suggestions];
  },
  mounted() {
    this.labelContainer = this.$refs["label-container"];
    if (this.inputValue) {
      this.handleLabelStyles();
    }
  },
  methods: {
    handleLabelStyles() {
      this.labelContainer.classList.add("floatlabel");
      this.$emit("focus");
    },
    removeLabelStyles(target) {
      if (target.value === "" && !this.inputValue) {
        this.labelContainer.classList.remove("floatlabel");
      }
      this.$emit("blur");
    },
    selectSuggestion(suggestion){
      this.labelContainer.classList.add("floatlabel");
      this.inputValue = suggestion;
      this.showSuggestions = false;
    },
    filterSuggestions(value) {
      this.filteredSuggestions = this.suggestions.filter(suggestion => suggestion.toLowerCase().includes(value.toLowerCase()));
    },
    hide() {
      this.showSuggestions = false;
    },
  },
};
</script>

<style lang="scss" scoped>
.floatlabel-input {
  position: relative;
  display: flex;
  padding: 5px;
  width: 100%;
  height: 40px;
  padding: 5px;
  border-radius: 10px;
  box-shadow: none;
  border: 1px solid #ced4da;
  background-color: white;
}
.floatlabel-input__content {
  position: relative;
  display: flex;
  flex-basis: 100%;
  height: 100%;
}
.floatlabel-input__label {
  position: absolute;
  align-items: center;
  padding-left: 2px;
  width: 85%;
  height: 100%;
  top: 9%;
  left: 0;
  pointer-events: none;
  transition: all 0.2s;
  font-size: 1rem;
  color: rgb(122, 122, 122);
  text-align: left;
}
.floatlabel-input__text-label {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  margin: 0px;
}

.floatlabel {
  top: -1.5rem;
  left: 0%;
  font-size: 0.75rem;
}

.floatlabel-input__input {
  width: 100%;
  height: 100%;
  border: none;
  border-radius: 10px;
  font-size: 1rem;
  background-color: white;
}
.floatlabel-input__input:focus {
  outline: 0;
}

.suggestions {
  position: absolute;
  top: 40px;
  left: 0px;
  width: 100%;
  height: auto;
  max-height: 150px;
  overflow-y: scroll;
  background-color: white;
  z-index: 10;
}

.suggestions-list {
  padding: 0;
  margin: 0;
  list-style-type: none;
  text-align: left;
  border: 1px solid rgba(196, 196, 196, 0.767);
}

.suggestions-list__item {
  padding: 5px 10px;
  cursor: pointer;
  &:hover {
    background-color: #e9ecef;
  }
}

.suggestions-list__item:hover {
  background-color: #e9ecef;
}

.suggestions::-webkit-scrollbar {
  width: 5px;
}
.suggestions::-webkit-scrollbar-track {
  box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
  border-radius: 10px;
}
.suggestions::-webkit-scrollbar-thumb {
  border-radius: 10px;
  box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.5);
}

input:disabled {
  background-color: white;
}
</style>
