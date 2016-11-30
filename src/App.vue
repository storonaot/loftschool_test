<template lang="jade">
  .container
    Logo
    .section(v-show="!flag")
      .flex-wrapper
        .block.is-wide
          h3.title.is-hovered(
            v-on:click="currentGroupId = null"
            v-bind:class="!currentGroupId ? 'is-active' : ''"
          ) Ваши друзья
          .inner
            ul.list(v-if="filteredByGroup.length")
              li.item.is-left(v-for="friend in filteredByGroup")
                .ava(v-bind:style="getPhoto(friend)")
                span {{ friend.first_name }} {{ friend.last_name }}
            .empty(v-else) Список пуст
          .bottom.is-center
            button.button(
              v-show="currentGroupId"
              v-on:click="updateGroup()"
            ) Редактировать список
        .block.is-narrow
          h3.title(v-on:click="currentGroupId = null") Списки друзей
          .inner
            ul.list(v-if="groups.length")
              li.item.is-hovered(
                v-for="group in groups"
                v-on:click="currentGroupId = group.gid"
                v-bind:class="currentGroupId === group.gid ? 'is-active' : ''"
              )
                span {{ cropString(group.name) }}
                .cross.is-delete.is-gray(v-on:click="deleteGroup")
            .empty(v-else) Создайте список
          .bottom
            button.button(
              v-on:click="createGroup"
            ) Создать список
  Overlay(
    v-if="flag"
    v-on:click="canselChanges"
  )
    Modal(
      v-if="flag"
      v-on:click.stop="() => {}"
    )
      .header
        span.modal-title Выберите друзей
        .cross.is-delete(v-on:click="canselChanges")
      .field-group
        input.field(
          type="text"
          placeholder="Начните вводить имя друга"
          v-model="searchValue"
        )
        input.field(
          v-if="currentModal === 'create'"
          type="text"
          v-model="newGroupName"
          placeholder="Название"
        )
        input.field(
          v-else
          type="text"
          v-model="currentName"
          v-bind:value="currentGroupName"
        )
      .flex-wrapper.is-modal
        .block
          h3.title Ваши друзья
          .inner(v-if="revFilteredByGroup.length")
            ul.list
              li.item.is-hovered(
                v-on:click="addFriendToGroup(friend.uid)"
                v-for="friend in revFilteredByGroup | filterBy searchValue in 'first_name' 'last_name'"
              )
                .about
                  .ava(v-bind:style="getPhoto(friend)")
                  span {{ friend.first_name }} {{ friend.last_name }}
                .cross.is-gray
          div(v-else) Все друзья были добавлены
        .block
          h3.title Друзья в списке
          .inner(v-if="filteredByGroup.length")
            ul.list
              li.item.is-hovered(
                v-for="friend in filteredByGroup"
                v-on:click="removeFriendFromGroup(friend.uid)"
              )
                .about
                  .ava(v-bind:style="getPhoto(friend)")
                  span {{ friend.first_name }} {{ friend.last_name }}
                .cross.is-delete.is-gray
          .empty(v-else) Добавьте друзей в список
      .footer
        div(v-if="currentModal==='update'")
          button.button(v-on:click="done") Сохранить
        button.button(
          v-if="newGroupName && newGroupName !== currentGroupName"
          v-on:click="saveNewGroup"
        ) Сохранить
</template>

<script lang="babel">
  import friendsList from './friends'

  const friends = friendsList.map((x) => ({
    uid: x.uid,
    last_name: x.last_name,
    first_name: x.first_name,
    photo_100: x.photo_100,
    groups: []
  }))

  export default {
    data () {
      return {
        flag: false,
        currentGroupId: null,
        currentModal: null,
        friends: friends,
        groups: [],
        seqGroupId: 1,
        newGroupName: null,
        searchValue: null,
        currentName: null,
      }
    },
    computed: {
      currentGroupName() {
        const name = this.groups.filter((x) => x.gid === this.currentGroupId)[0]
        return name ? name.name : 'Без названия'
      },
      filteredByGroup() {
        if (this.currentGroupId !== null) {
          const newArray = []
          this.friends.forEach((friend) => {
            friend.groups.filter((x) => x === this.currentGroupId).length > 0 ? newArray.push(friend) : null
          })
          return newArray
        }
        return this.friends
      },
      revFilteredByGroup() {
        if (this.currentGroupId !== null) {
          const newArray = []
          this.friends.forEach((friend) => {
            friend.groups.filter((x) => x === this.currentGroupId).length > 0 ? null : newArray.push(friend)
          })
          return newArray
        }
        return this.friends
      },
    },
    methods: {
      cropString(str) {
        if (str.length > 10) {
          return str.substring(0, 20) + '...'
        } else {
          return str
        }
      },
      done() {
        if (this.currentName !== this.currentGroupName) {
          this.groups.filter((x) => x.gid === this.currentGroupId)[0].name = this.currentName
        }
        this.flag = false
        this.searchValue = null
      },
      getPhoto(friend) {
        return `background-image: url("${friend.photo_100}")`
      },
      canselChanges() {
        this.currentGroupId = null
        this.flag = false
      },
      deleteGroup() {

        this.groups.forEach((group, index) => {
          if (group.gid === this.currentGroupId) {
            this.groups.splice(index, 1)
          }
        })

        this.friends.forEach(friend => {
          friend.groups.forEach((id, index) => {
            if (id === this.currentGroupId) {
              friend.groups.splice(index, 1)
            }
          })
        })

        this.currentGroupId = null
        this.flag = false

      },
      saveNewGroup() {
        this.groups.push({
          name: this.newGroupName,
          gid: ++this.seqGroupId,
        })

        this.friends.forEach(friend => {
          friend.groups.forEach((item, index) => {
            if (item === 0) {
              friend.groups.splice(index, 1, this.seqGroupId)
            }
          })
        })

        this.currentGroupId = this.seqGroupId
        this.flag = false
        this.newGroupName = null
      },
      createGroup() {
        this.flag = true
        this.currentModal = 'create'
        this.currentGroupId = 0
      },
      updateGroup() {
        this.flag = true
        this.currentModal = 'update'
      },
      addFriendToGroup(id) {
        const arr = this.friends.filter((x) => x.uid === id)[0].groups
        arr.push(this.currentGroupId)
      },
      removeFriendFromGroup(id) {
        const arr = this.friends.filter((x) => x.uid === id)[0].groups
        let place = null
        arr.forEach((item, index) => {
          if (item === this.currentGroupId) {
            place = index
          }
        })
        arr.splice(place, 1)
      },
    },
    components: {
      Overlay: require('./components/Overlay.vue'),
      Modal: require('./components/Modal.vue'),
      Logo: require('./components/Logo.vue'),
    }
  }
</script>

<style lang="sass?indentedSyntax=true" scoped>
  .container
    padding: 10vh 0
  .section
    max-height: calc(80vh - 58px)
    overflow: hidden
  .flex-wrapper
    display: flex
    width: 70%
    max-width: 650px
    margin: 0 auto
    border: 1px solid #ff7f50
    &.is-modal
      width: 100%
      border: 0
      height: calc(100% - 135px)
      overflow: hidden
      max-width: 100%
  .block
    position: relative
    width: 50%
    padding: 15px 0 15px 15px
    font-size: 12px
    font-weight: 500
    & + &
      border-left: 1px solid #f0f0f0
    &.is-wide
      width: 65%
    &.is-narrow
      width: 35%
  .inner
    position: relative
    min-width: 100%
    height: 300px
    overflow-y: auto
    padding-right: 15px
  .empty
    position: absolute
    top: 50%
    left: 50%
    transform: translate(-50%, -50%)
    font-size: 16px
    text-align: center
  .title
    margin: 0
    padding: 0 0 5px 0
    border-bottom: 1px solid #f0f0f0
    font-size: 12px
    cursor: default
    &.is-hovered
      cursor: pointer
      &:hover
        color: #ff7f50
    &.is-active
      cursor: default
      color: #ff7f50
  .ava
    display: inline-block
    width: 45px
    height: 45px
    margin-right: 5px
    border-radius: 50%
    background-size: cover
  .list
    margin: 0
    padding: 0
  .item
    display: flex
    align-items: center
    padding: 5px 5px
    border-bottom: 1px solid #f0f0f0
    justify-content: space-between
    cursor: default
    &.is-hovered:hover
      cursor: pointer
      background-color: #f0f0f0
    &.is-active
      background-color: #f0f0f0
      color: #ff7f50
    &.is-left
      justify-content: flex-start
  .button
    padding: 7px 15px
    border: 1px solid #ff7f50
    background-color: #ff7f50
    color: #fff
    font-size: 12px
    border-radius: 10px
    cursor: pointer
    &:hover
      background-color: #fff
      color: #ff7f50
    &:active
      outline: 0
  .bottom
    padding-top: 10px
    padding-right: 10px
    text-align: right
    &.is-center
      text-align: center
  .field-group
    display: flex
    padding: 0 15px
    height: 45px
    background-color: #f0f0f0
    align-items: center
    justify-content: space-between
  .field
    width: calc(50% - 40px)
    height: 25px
    padding: 0
    margin: 0
    border: 0
    border-radius: 13px
    font-size: 13px
    padding: 0 15px
    border: 1px solid #fff
    &:focus
      outline: 0
      border-color: #ff7f50
  .header, .footer
    height: 45px
    padding: 0 15px
    display: flex
    align-items: center
    justify-content: space-between
  .header
    background-color: #ff7f50
  .footer
    background-color: #f0f0f0
    justify-content: flex-end
  .modal-title
    line-height: 45px
    color: #fff
  .cross
    position: relative
    width: 20px
    height: 20px
    cursor: pointer
    &::before, &::after
      content: ''
      display: inline-block
      position: absolute
      background-color: #fff
      border-radius: 4px
    &::before
      width: 4px
      height: 15px
      left: 50%
      top: 50%
      transform: translate(-50%, -50%)
    &::after
      height: 4px
      width: 15px
      left: 50%
      top: 50%
      transform: translate(-50%, -50%)
    &.is-delete
      transform: rotate(45deg)
    &:hover
      &::before, &::after
        background-color: #c4c4c4
    &.is-gray
      &::before, &::after
        background-color: #c4c4c4
  .about
    display: flex
    align-items: center
</style>
