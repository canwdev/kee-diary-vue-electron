<template>
  <q-tree
      class="group-tree-wrap"
      :nodes="groupTree"
      node-key="id"
      label-key="name"
      selected-color="primary"
      :selected.sync="selectedGroupUuidStr"
      default-expand-all
  >
    <template v-slot:default-header="prop">
      <div class="row items-center">
        <q-avatar
            class="tree-icon"
            size="32px" square>
          <img :src="icons[prop.node.iconIndex]">
        </q-avatar>
        <div class="tree-name">{{ prop.node.name }}</div>
      </div>
    </template>
  </q-tree>
</template>

<script>
import icons from "@/assets/db-icons"
import store from "@/store"
import {getGroupTree} from "@/utils/kdbx-utils"

export default {
  name: "GroupTree",
  props: {
    selectedGroupUuid: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      icons: Object.freeze(icons.items),
      groupTree: [],
      selectedGroupUuidStr: null,
    }
  },
  computed: {
    database: {
      get: () => store.getters.database
    },
    groupUuidMap() {
      const groupTree = this.groupTree

      function getMap(node, counter = 0, map = {}) {
        if (!node || node.length === 0) return map

        node.forEach(item => {
          const children = item.children
          const uuidStr = item.uuid.id
          map[uuidStr] = item.uuid

          getMap(children, counter + 1, map)
        })

        return map
      }

      return getMap(groupTree)
    },
    mSelectedGroupUuid: {
      get() {
        return this.selectedGroupUuid
      },
      set(nv) {
        this.$emit('update:selectedGroupUuid', nv)
      }
    }
  },
  watch: {
    database: {
      handler(db) {
        if (!db) {
          this.groupTree = []
          return
        }

        this.groupTree = getGroupTree(db.groups)

        // re-open last group
        if (this.mSelectedGroupUuid) {
          this.selectedGroupUuidStr = this.mSelectedGroupUuid.id
          return
        }
        if (this.groupTree[0]) {
          this.selectedGroupUuidStr = this.groupTree[0].id
        }
      },
      immediate: true
    },
    selectedGroupUuidStr: {
      handler(nv) {
        if (!nv) {
          this.mSelectedGroupUuid = null
          return
        }
        this.mSelectedGroupUuid = this.groupUuidMap[this.selectedGroupUuidStr]
        // this.handleRefreshEntryList()
      },
      immediate: true
    },
  }
}
</script>

<style lang="stylus" scoped>
.group-tree-wrap {
  .tree-icon {
    border-radius 0
    margin-right: 8px
  }

  >>> .q-tree__node--selected {
    background $primary
  }

  .text-primary {
    .tree-name {
      color #fff
      font-weight: bold;
    }
  }
}

</style>