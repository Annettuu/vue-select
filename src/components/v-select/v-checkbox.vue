<template>
    <input
      type="checkbox"
      :checked="checked"
      class="v-checkbox"
    >
  </template>
  
  <script setup>
  </script>
  
  <style lang="scss">
  .v-checkbox[type=checkbox] {
    position: relative;
    border: 1px solid #D7D7D7;
    border-radius: 4px;
    background-color: white;
    cursor: pointer;
    line-height: 0;
    margin: 0;
    margin-right: 8px;
    outline: 0;
    padding: 0;
    vertical-align: sub;
    height: 16px;
    width: 16px;
    -webkit-appearance: none;
    &:before {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      display: block;
      content: '';
      width: 16px;
      height: 16px;
      background: url(src/assets/images/svg/active-checkbox.svg) no-repeat center;
      opacity: 0;
    }
    &:checked:before {
      opacity: 1;
    }
  }
  </style>