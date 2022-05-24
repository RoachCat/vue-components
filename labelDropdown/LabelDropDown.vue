<template>
  <div
    v-click-outside="hide"
    @click="openDropdown"
    ref="dropdown-container"
    class="dropdown-container"
    :title="title"
  >
    <div class="dropdown-container__selected-option">
      <div ref="label-container" class="label-container">
        <p class="label-container__text">{{ label }}</p>
      </div>
      <p ref="dropdown-container__text" class="dropdown-container__text">
        {{ selectedOptionLabel }}
      </p>
      <div class="dropdown-container__icon-container">
        <button
          v-if="deleteButton && selectedOption"
          @click.stop="deleteSelectedOption"
          class="delete-button"
        >
          X
        </button>
        <i class="dropdown-container__icon ion ion-chevron-down-outline"></i>
      </div>
    </div>
    <div
      v-if="!dropdownCollapsed"
      class="dropdown-container__options-container"
      ref="options-container"
    >
      <slot name="dropdown-option">
        <div
          @click="selectOption(option)"
          v-for="(option, index) in options"
          :key="index"
        >
          <p class="dropdown-container__option">{{ option.label }}</p>
        </div>
      </slot>
    </div>
  </div>
</template>

<script>
export default {
  name: "FloatDropdown",
  props: {
    options: Array,
    label: String,
    floatLabel: {
      type: Boolean,
      default: false,
    },
    deleteButton: {
      type: Boolean,
      default: false,
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
      dropdownCollapsed: true,
      selectedOptionLabel: null,
      selectedOption: null,
      labelDomElement: null,
      optionsContainerDomElement: null,
      textDomElement: null,
    };
  },
  computed: {
    title() {
      if (this.selectedOption) {
        return `${this.label}: ${this.selectedOptionLabel}`;
      }
      return this.label;
    },
  },
  watch: {
    selectedOption() {
      if (this.selectedOption) {
        if (this.floatLabel) {
          this.labelDomElement.classList.add("floatlabel");
        } else {
          this.labelDomElement.style.visibility = "hidden";
        }
        this.textDomElement.style.color = "#043663";
      } else {
        this.labelDomElement.classList.remove("floatlabel");
        this.textDomElement.style.color = "rgb(122, 122, 122)";
      }
    },
  },
  created() {
    this.selectedOption = this.$attrs.value;
    this.selectedOptionLabel = this.$attrs.value;
  },
  mounted() {
    // eslint-disable-next-line operator-linebreak
    this.labelDomElement = this.$refs["label-container"];
    this.optionsContainerDomElement = this.$refs["options-container"];
    this.textDomElement = this.$refs["dropdown-container__text"];
  },
  methods: {
    selectOption(option) {
      if (option.label !== this.selectedOptionLabel) {
        this.selectedOptionLabel = option.label;
        this.selectedOption = option.label;
        this.$emit("change", option);
        this.$emit("input", option.label);
      }
      this.dropdownCollapsed = true;
    },
    deleteSelectedOption() {
      this.selectedOption = null;
      this.selectedOptionLabel = null;
      this.$emit("change", null);
      this.$emit("input", null);
    },
    openDropdown() {
      this.dropdownCollapsed = !this.dropdownCollapsed;
    },
    hide() {
      this.dropdownCollapsed = true;
    },
  },
};
</script>

<style lang="scss" scoped>
.dropdown-container {
  position: relative;
  height: 40px;
  border: 1px solid #ced4da;
  border-radius: 10px;
  cursor: pointer;
}
.label-container {
  position: absolute;
  top: 0.5rem;
  left: 0.5rem;
  width: 80%;
  pointer-events: none;
  transition: all 0.2s;
  font-size: 1rem;
  color: rgb(122, 122, 122);
  text-align: left;
}
.label-container__text {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  margin: 0px;
}
.dropdown-container__selected-option {
  position: relative;
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  max-height: 40px;
  padding: 0.5rem;
  background-color: white;
  border: none;
  border-radius: 10px;
  font-size: 1rem;
}
.dropdown-container__text {
  margin: 0px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  color: black;
  cursor: pointer;
}
.floatlabel {
  top: -1.2rem;
  left: 1.4%;
  font-size: 0.75rem;
}
.delete-button {
  border: none;
  background: none;
}
.dropdown-container__icon-container {
  display: flex;
  align-items: baseline;
}
.dropdown-container__options-container {
  position: absolute;
  top: 100%;
  max-height: 150px;
  width: 100%;
  background-color: white;
  border: 1px solid rgba(196, 196, 196, 0.767);
  border-radius: 3px;
  overflow-x: hidden;
  z-index: 10;
}

.dropdown-container__options-container::-webkit-scrollbar {
  width: 5px;
}
.dropdown-container__options-container::-webkit-scrollbar-track {
  box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
  border-radius: 10px;
}
.dropdown-container__options-container::-webkit-scrollbar-thumb {
  border-radius: 10px;
  box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.5);
}

.dropdown-container__option {
  margin: 0px;
  padding: 5px 0px;
  cursor: pointer;
  text-align: center;
  font-size: 1rem;
}
.dropdown-container__option:hover {
  background-color: #e9ecef;
}
</style>
