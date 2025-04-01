<script lang="ts" setup>
import { ref } from 'vue';

const value = ref<number>(0);
const inputValue = ref<string>('0');
const unit = ref<string>('%');
const mouseEnterButton = ref<boolean>(false);
const isFocusInput = ref<boolean>(false)

const handleUnit = (type: string = 'px') => {
  unit.value = type;
  if (unit.value === '%' && value.value > 100) {
    value.value = 100;
    inputValue.value = '' + value.value
  }
}

const handleStepValue = (increase: boolean) => {
  if (unit.value === '%') {
    if ((value.value === 100) && increase){
      return
    }

    if ((value.value === 0) && !increase) {
      return
    }
  }

  value.value = increase ? value.value + 1 : value.value - 1;

  if (unit.value === '%' && value.value > 100) {
    value.value = 100
  }

  if (unit.value === '%' && value.value < 0) {
    value.value = 0
  }

  inputValue.value = '' + value.value
}

const onMouseEnter = () => {
  mouseEnterButton.value = true
}

const onMouseLeave = () => {
  mouseEnterButton.value = false
}

const handleFocusInput = () => {
  isFocusInput.value = true
}

const handleChangeValue = () => {
  isFocusInput.value = false
  inputValue.value = '' + formatValue(inputValue.value)
  value.value = formatValue(inputValue.value)
}

function isNumeric(str: string): boolean {
  return !isNaN(parseFloat(str)) && isFinite(Number(str));
}

const formatValue = (val: string) => {
  if (isNumeric(val) && !val.toString().includes(',')) {
    if (unit.value === '%' && val) {
      if (parseFloat(val) > 100) {
        return 100
      }
    }

    if (parseFloat(val) < 0) {
      return 0
    }

    return parseFloat(val)
  }

  let newValue = ''
  const splitValue = val.toString().split('')
  for (let index = 0; index < splitValue.length; index++) {
    if (splitValue[index] === ',' && index !== 0 && !newValue.includes('.') && isNumeric(splitValue[index+1])) {
      newValue += '.'
    }


    if (isNumeric(splitValue[index])) {
      newValue += splitValue[index]
    }

    if ((!isNumeric(splitValue[index]) && splitValue[index] !== ',')  && index !== 0 && newValue) {
      break;
    }
  }

  if (newValue === '') {
    return 0
  }

  if (parseFloat(newValue) > 100 && unit.value === '%') {
    return 100
  }

  return parseFloat(newValue)
}

</script>

<template>
  <div class="text-xs wd-length flex flex-col gap-y-4">
    <div class="flex items-center gap-2 h-[36px] wd-length__container">
      <div class="label grow max-w-[100%]">Unit</div>
      <div class="control p-0.5 rounded-lg h-full flex flex-wrap">
        <button :class="['w-1/2 flex items-center rounded-l-lg justify-center hover:cursor-pointer', {
          'selected': unit === '%',
        }]" @click="handleUnit('%')">%</button>
        <button :class="['w-1/2 flex items-center rounded-r-lg justify-center hover:cursor-pointer', {
          'selected': unit === 'px',
        }]" @click="handleUnit('px')">px</button>
      </div>
    </div>
    <div class="flex items-center gap-2 font-xs h-[36px] wd-length__container">
      <div class="label grow max-w-[100%]">Value</div>
      <div :class="['control h-full rounded-lg inline-flex',{
        'is-hover': !mouseEnterButton,
        'focus-input': isFocusInput
      }]">
        <button :class="['decrease w-[36px] flex items-center rounded-l-lg justify-center', {
           'disabled cursor-not-allowed hover:cursor-not-allowed': value === 0,
           'hover:cursor-pointer': value !== 0,
        }]" @click="handleStepValue(false)"
                @mouseenter="onMouseEnter"
                @mouseleave="onMouseLeave">-</button>
        <!--- why don't set type input is number? input 123a -> 123 :(((  --->
        <input v-model="inputValue" class="max-w-[68px] grow text-center flex items-center justify-center focus:outline-none"
               @focus="handleFocusInput"
               @blur="handleChangeValue"
        />
        <button :class="['increase w-[36px] flex items-center justify-center rounded-r-lg', {
           'disabled cursor-not-allowed hover:cursor-not-allowed': unit === '%' && value === 100,
           'hover:cursor-pointer': unit === '%' && value !== 100,
        }]" @click="handleStepValue(true)"
                @mouseenter="onMouseEnter"
                @mouseleave="onMouseLeave">+</button>
      </div>
    </div>
  </div>
</template>