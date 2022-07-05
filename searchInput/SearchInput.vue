<template>
  <div class="search-input" v-click-outside="hide">
    <div class="search-input__icon" v-if="icon">
      <slot name="icon" class="icon" ref="icon-slot"></slot>
    </div>
    <div class="search-input__main-content">
      <input
        class="search-input__input"
        min="0"
        @blur="removeLabelStyles($event.target)"
        @focus="handleLabelStyles($event.target)"
        @input="handleText($event)"
        v-model="inputValue"
        :type="type"
        :title="title"
        :disabled="disabled"
        :placeholder="!floatLabel ? label : ''"
      />
      <div v-if="floatLabel" class="label" ref="label-container">
        <p class="label__text" htmlFor="">{{ label }}</p>
      </div>
      <div v-if="!dropdownCollapsed && sections.length" class="dropdown">
        <slot name="main-action"></slot>
        <div v-for="(section, index) in sections" :key="'section-' + index">
          <template v-if="section.options.length">
            <slot name="section-title" :section="section">
              <p v-if="section.sectionName" class="dropdown__section-name">
                {{ section.sectionName }}
              </p>
            </slot>
            <div
              v-for="(option, index) in section.options"
              :key="'option-' + index"
              @click="selectOption(option)"
            >
              <slot name="dropdown-option" :option="option">
                <p class="dropdown__option">{{ option.label }}</p>
              </slot>
            </div>
            <hr v-if="index < sections.length - 1" class="dropdown__divider" />
          </template>
        </div>
      </div>
      <div class="search-input__spinner" v-if="loading">
        <div class="search-input__spinner-icon"></div>
      </div>
    </div>
    <div class="search-input__dropdown-button-container" v-if="dropdown">
      <button class="search-input__dropdown-button" @click="toggleDropdown">
        V
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: "FloatSearchInput",
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
  props: {
    field: {
      type: String,
    },
    forceSelection: {
      type: Boolean,
      default: false,
    },
    floatLabel: {
      type: Boolean,
      default: false,
    },
    dropdown: {
      type: Boolean,
      default: false,
    },
    delay: {
      type: Number,
      default: 200,
    },
    icon: {
      type: Boolean,
      default: false,
    },
    lengthRequest: {
      type: Number,
      default: 1,
    },
    suggestions: {
      type: Array,
      default: () => [],
    },
    label: {
      type: String,
      required: false,
    },
    type: {
      type: String,
      default: "text",
    },
    disabled: {
      default: false,
      type: Boolean,
      required: false,
    },
  },
  data() {
    return {
      labelContainer: null,
      mainContent: null,
      inputValue: "",
      selectedOption: null,
      sections: [],
      timeout: null,
      dropdownCollapsed: true,
      loading: false,
    };
  },
  watch: {
    suggestions: {
      deep: true,
      handler() {
        this.sections = [...this.suggestions];
        this.dropdownCollapsed = false;
        this.loading = false;
      },
    },
    inputValue(value) {
      if (value) {
        this.handleLabelStyles();
      }
    },
    $attrs: {
      immediate: true,
      handler() {
        if (this.$attrs.value && this.field in this.$attrs.value) {
          this.selectedOption = this.$attrs.value;
          this.inputValue = this.$attrs.value[this.field];
        }
      },
    },
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
    this.sections = [...this.suggestions];
  },
  mounted() {
    this.labelContainer = this.$refs["label-container"];
    if (this.inputValue) {
      this.handleLabelStyles();
    }
  },
  methods: {
    handleText(event) {
      if (this.selectedOption) {
        this.selectedOption = null;
      }
      this.loading = false;
      clearTimeout(this.timeout);
      if (event.target.value.length >= this.lengthRequest) {
        this.loading = true;
      }
      this.timeout = setTimeout(() => this.emitInputValue(event), this.delay);
    },
    emitInputValue(event) {
      this.$emit("search", { originalEvent: event, query: event.target.value });
      this.$emit("input", null);
    },
    selectOption(option) {
      this.inputValue = option.label;
      this.selectedOption = option;
      this.$emit("input", option);
      this.$emit("selectOption", option);
    },
    hide() {
      this.dropdownCollapsed = true;
    },
    toggleDropdown() {
      this.dropdownCollapsed = !this.dropdownCollapsed;
    },
    handleLabelStyles() {
      this.dropdownCollapsed = true;
      if (this.floatLabel) {
        this.labelContainer.classList.add("floatlabel");
      }
    },
    async removeLabelStyles(target) {
      await this.sleep(100);
      if (this.forceSelection) {
        if (!this.selectedOption) {
          this.inputValue = "";
          this.sections = [];
          this.$emit("input", null);
          target.value = "";
        }
      }
      if (this.floatLabel) {
        if (target.value === "") {
          this.labelContainer.classList.remove("floatlabel");
        }
      }
      this.$emit("blur");
    },
    sleep(milliseconds) {
      return new Promise((resolve) => setTimeout(resolve, milliseconds));
    },
  },
};
</script>

<style lang="scss" scoped>
.search-input {
  display: flex;
  position: relative;
  width: 100%;
  height: 40px;
  padding: 5px;
  border-radius: 10px;
  box-shadow: none;
  border: 1px solid #ced4da;
  background-color: white;
}

.search-input__main-content {
  position: relative;
  display: flex;
  flex-basis: 100%;
  height: 100%;
}

.search-input__icon {
  margin-right: 2px;
}

.label {
  position: absolute;
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

.floatlabel {
  top: -1.5rem;
  left: 0%;
  font-size: 0.75rem;
  & p {
    overflow: initial;
  }
}
.label__text {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.search-input__input {
  width: 100%;
  height: 100%;
  border: none;
  border-radius: 10px;
  font-size: 1rem;
  background-color: white;
}

.search-input__input:focus {
  outline: 0;
}

.search-input__spinner {
  color: #737373;
  position: absolute;
  top: 11%;
  right: 2%;
}

.search-input__spinner-icon {
  height: 16px;
  width: 16px;
  color: #5a5a5a;
  position: relative;
  display: inline-block;
  border: 3px solid;
  border-radius: 50%;
  border-top-color: transparent;
  animation: rotate 1s linear infinite;
}

.search-input__dropdown-button-container {
  display: flex;
  justify-content: center;
  border-left: 1px solid #ced4da;
  width: 32px;
}
.search-input__dropdown-button {
  background: none;
  border: none;
  transform: scale(2, 1);
  font-size: 12px;
  color: #8b8b8b;
}

.dropdown {
  position: absolute;
  top: 31px;
  max-height: 250px;
  width: 97%;
  background-color: white;
  border: 1px solid rgba(196, 196, 196, 0.767);
  border-radius: 3px;
  overflow-x: hidden;
  z-index: 10;
}

.dropdown::-webkit-scrollbar {
  width: 5px;
}
.dropdown::-webkit-scrollbar-track {
  box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
  border-radius: 10px;
}
.dropdown::-webkit-scrollbar-thumb {
  border-radius: 10px;
  box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.5);
}

.dropdown__section-name {
  margin: 15px 0px 0px 20px;
  font-size: 0.9rem;
  text-align: left;
}
.dropdown__divider {
  margin: 1rem 20px 0px 20px;
}
.dropdown__option {
  margin: 0px;
  margin-left: 20px;
  padding: 7px 0px;
  cursor: pointer;
  text-align: left;
  font-size: 1rem;
}

.dropdown__option:hover {
  background-color: #e9ecef;
}

input:disabled {
  background-color: white;
}
</style>
