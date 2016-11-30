<template lang="jade">
  .logo
    span другофильтр
    div.round
</template>

<style lang="sass?indentedSyntax=true" scoped>
  .logo
    position: relative
    display: flex
    flex-direction: column
    align-items: center
    max-width: 200px
    margin: 0 auto 25px auto
    font-family: 'MarckScript'
    font-size: 27px
    color: #ffa16c
  .round
    position: absolute
    bottom: -5px
    right: 91px
    display: flex
    width: 5px
    height: 5px
    border-radius: 50%
    border: 2px solid #ffa16c
    box-sizing: content-box
    &::before, &::after
      position: absolute
      top: 2px
      content: ''
      background-color: #ffa16c
      height: 2px
      width: 20px
      border-radius: 10%
    &::before
      right: 9px
      width: 31px
    &::after
      left: 9px
      width: 23px
</style>
