<script>
export default {
  props: {
    label: {
      type: String,
      required: true
    },
    type: {
      type: String,
      default: 'text'
    },
    value: {
      type: String,
      default: ''
    },
    placeholder: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      localStorageValue: window.localStorage.getItem(this.label) || this.value
    };
  },
  methods: {
    handleInput({target}) {
      const newValue = target.value;
      window.localStorage.setItem(this.label, newValue);
      this.localStorageValue = newValue;
    }
  }
}
</script>
<template>
  <div class="relative h-10 input-component mb-5">
    <input
      :id="label"
      :type="type"
      :name="label"
      :value="localStorageValue"
      :placeholder="placeholder"
      @input="handleInput"
      class="h-full w-full border-blue-10 px-2 transition-all border-blue rounded-md bg-zinc-900 text-blue-100"
    />
    <label :for="label" class="absolute left-2 transition-all bg-blue-100 rounded px-1">
      {{ label }}
    </label>
  </div>
</template>



<style scoped>
label {
  top: 0%;
  transform: translateY(-50%);
  font-size: 11px;
  color: rgba(37, 99, 235, 1);
}
.empty input:not(:focus) + label {
  top: 50%;
  transform: translateY(-50%);
  font-size: 14px;
}
input:not(:focus) + label {
  color: rgba(70, 70, 70, 1);
}
input {
  border-width: 1px;
}
input:focus {
  outline: none;
  border-color: rgba(37, 99, 235, 1);
}
</style>, watchwatch, 