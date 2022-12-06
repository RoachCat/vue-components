<template>
  <div class="floatlabel-input">
    <div class="floatlabel-input__content">
      <input
        class="floatlabel-input__input"
        :type="type"
        min="0"
        @blur="removeLabelStyles($event.target)"
        @focus="handleLabelStyles($event.target)"
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
    }
  },
  data() {
    return {
      inputValue: "",
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
      if (target.value === "") {
        this.labelContainer.classList.remove("floatlabel");
      }
      this.$emit("blur");
    },
  },
};
</script>

<style lang="scss" scoped>
.floatlabel-input {
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

input:disabled {
  background-color: white;
}
</style>
