<template>
  <div class="folders">
    <SidebarList title="Folders">
      <DraggableTree
        ref="tree"
        :data="localFolders"
        :indent="10"
        :space="0"
        :draggable="true"
        @change="onTreeChange"
      >
        <template v-slot="{ data, store }">
          <div
            v-if="!data.isDragPlaceHolder"
            @dragover.prevent
            @drop="onDropTreeNode($event, data.id)"
          >
            <SidebarListItem
              :id="data.id"
              :title="data.name"
              :children="!!data.children.length"
              :open="data.open"
              @click:toggle="onNodeToggle(data, store)"
            />
          </div>
        </template>
      </DraggableTree>
      <template v-slot:action>
        <AppIcon
          name="plus"
          @click="onAddFolder"
        />
      </template>
    </SidebarList>
  </div>
</template>

<script>
import SidebarList from './SidebarList.vue'
import SidebarListItem from './SidebarListItem.vue'
import { mapGetters } from 'vuex'
import cloneDeep from 'lodash-es/cloneDeep'
import { DraggableTree } from 'vue-draggable-nested-tree/dist/vue-draggable-nested-tree'

export default {
  name: 'TheFolders',

  components: {
    SidebarList,
    SidebarListItem,
    DraggableTree
  },

  data () {
    return {
      localFolders: [],
      draggable: true,
      editableFolderId: null
    }
  },

  computed: {
    ...mapGetters('folders', ['folders']),
    tree () {
      return this.$refs.tree
    }
  },

  async created () {
    this.localFolders = cloneDeep(this.folders)
    this.$watch('folders', () => {
      this.localFolders = cloneDeep(this.folders)
    })
  },

  methods: {
    async onAddFolder () {
      this.$store.dispatch('folders/addFolder')
    },
    onDropTreeNode (e, folderId) {
      const data = e.dataTransfer.getData('payload')
      if (data) {
        const id = JSON.parse(data).value
        const payload = {
          $set: { folderId }
        }
        this.$store.dispatch('snippets/updateSnippet', { id, payload })
      }
    },
    onTreeChange (node, newTree, oldTree) {
      const folders = newTree.getPureData()
      this.$store.dispatch('folders/updateFolders', folders)
    },
    onNodeToggle (data, store) {
      store.toggleOpen(data)
      const folders = this.tree.getPureData()
      this.$store.dispatch('folders/updateFolders', folders)
    }
  }
}
</script>

<style lang="scss" scoped>
.folders {
}
</style>
