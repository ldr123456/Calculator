<template>
  <div class="calculator">
    <input
      type="text"
      v-model="inputValue"
      placeholder="0"
      disabled="disabled"
      ref="one"
    >
    <div
      class="container"
      @click="handleBtnClick"
    >
      <button class="NumBtn NumBtn-1">AC</button>
      <button class="NumBtn NumBtn-2">C</button>
      <button class="NumBtn NumBtn-3">B</button>
      <button class="NumBtn NumBtn-4">/</button>
      <button class="NumBtn NumBtn-5">7</button>
      <button class="NumBtn NumBtn-6">8</button>
      <button class="NumBtn NumBtn-7">9</button>
      <button class="NumBtn NumBtn-8">*</button>
      <button class="NumBtn NumBtn-9">4</button>
      <button class="NumBtn NumBtn-10">5</button>
      <button class="NumBtn NumBtn-11">6</button>
      <button class="NumBtn NumBtn-12">-</button>
      <button class="NumBtn NumBtn-13">1</button>
      <button class="NumBtn NumBtn-14">2</button>
      <button class="NumBtn NumBtn-15">3</button>
      <button class="NumBtn NumBtn-16">+</button>
      <button class="NumBtn NumBtn-17">0</button>
      <button class="NumBtn NumBtn-18">.</button>
      <button class="NumBtn NumBtn-19">=</button>
    </div>
  </div>
</template>

<script setup>
const { ref } = require('@vue/reactivity')
// 定义变量
const inputValue = ref('') // 输入框内容
const ClickNumberContent = ref('') // 存储拼接的元素
const record = ref(false) // 区分操作数还是操作符，操作数为false
const lastBtn = ref([]) // 定义数组，存储每个元素对应的 record
const flag = ref(false) // 判断是否进行过运算
const result = ref([]) // 存储第一次结算的结果
const handleBtnClick = (e) => {
  const ClickNumber = e.target.innerHTML
  if (!isNaN(ClickNumber)) {
    ClickNumberContent.value += ClickNumber
    inputValue.value = ClickNumberContent.value
    if (!isNaN(ClickNumberContent.value)) {
      const re = /^[0]\d/
      if (re.test(ClickNumberContent.value)) {
        ClickNumberContent.value = ClickNumberContent.value.substring(1)
        inputValue.value = ClickNumberContent.value
      }
    } else {
      let re = /\d+(\.\d+)?$/
      const lastNum = ClickNumberContent.value.match(re)
      const index = lastNum.index
      // 只有整数才能进入下面的条件语句，会忽略小数
      re = /^[0]\d/
      if (re.test(lastNum[0])) {
        ClickNumberContent.value = ClickNumberContent.value.substring(0, index) + ClickNumberContent.value.substring(index + 1)
        inputValue.value = ClickNumberContent.value
      }
    }
  }
  // 小数计算
  // 当 ClickNumberContent 不为空时，三种情况：
  if (ClickNumber === '.') {
    // 如果之前有数
    if (ClickNumberContent.value) {
      // 如果之前不存在小数点且存在数字，那么直接加小数点
      if (!isNaN(ClickNumberContent.value) && ClickNumberContent.value.indexOf('.') === -1) {
        ClickNumberContent.value += ClickNumber
        inputValue.value = ClickNumberContent.value
      } else if (isNaN(ClickNumberContent.value) && lastBtn.value[lastBtn.value.length - 2]) {
        // 如果前面的内容不是纯数字，且前一位是操作符
        ClickNumberContent.value += '.'
        inputValue.value = ClickNumberContent.value
      } else if (isNaN(ClickNumberContent.value) && !lastBtn.value[lastBtn.value.length - 2]) {
        // 如果之前的内容不是纯数字，且前一位是数，则取 ClickNumberContent 最后一个数字或小数
        const re1 = /\d+(\.\d+)?$/
        const lastNum1 = ClickNumberContent.value.match(re1)
        if (lastNum1[0].indexOf('.')) {
          ClickNumberContent.value += ClickNumber
          inputValue.value = ClickNumberContent.value
        }
      }
    } else {
      // 如果是直接点击小数点
      ClickNumberContent.value = '0.'
      inputValue.value = ClickNumberContent.value
    }
  }
  if (ClickNumber === '+' || ClickNumber === '-' || ClickNumber === '*' || ClickNumber === '/') {
    record.value = true
    // 如果之前没有输入操作数，则第一个操作数为默认的0，所以需要将ClickNumberContent补一个0进去
    if (!ClickNumberContent.value) {
      ClickNumberContent.value = '0'
    }
    // 如果上一个输入元素也为操作符（即连续点击运算符是）。则使用后输入的操作符
    if (lastBtn.value[lastBtn.value.length - 1]) {
      ClickNumberContent.value = ClickNumberContent.value.substring(0, ClickNumberContent.value.length - 1)
    }
    // 将每次获取的元素逐一拼接在ClickNumberContent中
    ClickNumberContent.value += ClickNumber
    // 将获得的元素显示在“屏幕”中
    inputValue.value = ClickNumberContent.value
  }
  // 计算结果，并储存
  if (ClickNumber === '=') {
    inputValue.value = calculate(ClickNumberContent.value) + ''
    // console.log(inputValue.value)
    // 将得到的数据存入 result
    result.value.push(inputValue.value)
    flag.value = true
    // 记录上一次的 ClickNumber
    lastBtn.value = lastBtn.value.splice(lastBtn.value.length - 1, 1)
  }
  // 如果存在计算结果
  if (flag.value) {
    // 如果点击运算符
    if (ClickNumber === '+' || ClickNumber === '-' || ClickNumber === '*' || ClickNumber === '/') {
      // 将式子的结果后面加上符号
      ClickNumberContent.value = result.value[result.value.length - 1] + ClickNumber
      inputValue.value = ClickNumberContent.value
      flag.value = false
    }
    // 如果直接点击数字，那么开启新一轮的运算
    if (!isNaN(ClickNumber)) {
      console.log(ClickNumber)
      ClickNumberContent.value = ClickNumber
      inputValue.value = ClickNumberContent.value
      flag.value = false
    }
    // 如果直接点小数，就是新一轮运算
    if (ClickNumber === '.') {
      ClickNumberContent.value = '0.'
      inputValue.value = ClickNumberContent.value
      flag.value = false
    }
  }
  // 区别存储每个输入的元素种类
  lastBtn.value.push(record.value)
  // 将record还原
  if (record.value) record.value = false
  // console.log(ClickNumberContent.value)
  // console.log(lastBtn.value)
}
const calculate = (NumberContent) => {
  // 先判断加减，在判断乘除
  // 获得加号的位置
  let index = NumberContent.indexOf('+')
  // 加号存在，则这个判断句中进行逐层递归
  if (index > -1) {
    return (calculate(NumberContent.substring(0, index)) + calculate(NumberContent.substring(index + 1)))
  }
  // 当式子没有加号时，会到这里
  index = NumberContent.lastIndexOf('-')
  if (index > -1) {
    return (calculate(NumberContent.substring(0, index)) - calculate(NumberContent.substring(index + 1)))
  }
  // 乘号
  index = NumberContent.indexOf('*')
  if (index > -1) {
    return (calculate(NumberContent.substring(0, index)) * calculate(NumberContent.substring(index + 1)))
  }
  // 除号
  index = NumberContent.lastIndexOf('/')
  if (index > -1) {
    return (calculate(NumberContent.substring(0, index)) / calculate(NumberContent.substring(index + 1)))
  }
  // 转换结果
  return Number(NumberContent)
}
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
}
.calculator {
  width: 340px;
  margin: 50px auto;
}
.calculator input {
  box-sizing: border-box;
  border-radius: 2%;
  padding: 0 6px;
  width: 340px;
  height: 60px;
  font-size: 32px;
  color: rgb(48, 42, 42);
  background: #ccc;
  text-align: right;
  outline: none;
  text-indent: -200px;
}
::-webkit-input-placeholder {
  color: rgb(48, 42, 42);
}
.calculator .container {
  display: grid;
  grid-template-rows: repeat(5, 65px); /* 五行 高度65px */
  grid-template-columns: repeat(4, 85px); /* 四列 宽度85px */
  width: 50%;
}
.NumBtn {
  color: rgb(48, 42, 42);
  text-align: center;
  border: 1px solClickNumber #bbb;
  outline: none;
  font-size: 33px;
  border-radius: 10%;
  background: #ccc;
}
.NumBtn-17 {
  grid-column-start: span 2; /* 数字 0 占据两列 */
}
</style>
