<template lang="jade">
  .modal
    slot
</template>

<script lang="babel">
  export default {}
</script>

<style lang="sass?indentedSyntax=true" scoped>
  .modal
    position: fixed
    top: 50%
    right: auto
    bottom: auto
    left: 50%
    width: 70%
    height: 70%
    transform: translate(-50%,-50%)
    background: #fff
    overflow: hidden
    border-radius: 10px
</style>
