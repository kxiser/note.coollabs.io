<template>
  <div>
    <div
      v-if="!$store.state.loading.localNotes"
      class="notes transition"
    >
      <div
        v-if="$store.state.notes.length === 0 && !$store.state.loading.remoteNotes"
        class="absolute w-full text-base font-bold text-center text-black centered transition"
      >
        No notes found! Let's create your first one!
        <div
          class="flex flex-col items-center justify-center h-full my-3 animated jello"
        >
          <PlusIcon
            size="60"
            class="plus"
            @click="addNewNote()"
          />
        </div>
      </div>
      <div v-if="$store.state.notes.length === 0 && $store.state.loading.remoteNotes">
        <div class="text-base font-bold text-center centered lg:text-4xl">
          <span>Loading</span>
          <span class="animated fadeIn">.</span>
          <span class="animated fadeIn delay-1s">.</span>
          <span class="animated fadeIn delay-2s">.</span>
        </div>
      </div>
      <note
        v-for="(note,index) in $store.state.notes"
        v-show="(!note.deletedLocally) && (!$store.state.selectedTag || (note.tags && note.tags.length > 0 && note.tags.includes($store.state.selectedTag))) && (note.title && note.title.match(new RegExp(`.*${$store.state.search}.*`,'giu')) || note.description && note.description.match(new RegExp(`.*${$store.state.search}.*`,'giu')) || (note.tags && note.tags.toString().match(new RegExp(`.*${$store.state.search}.*`,'giu')))|| (note.todo && note.todo.length > 0 && note.todo.map(todo => todo.line).toString().match(new RegExp(`.*${$store.state.search}.*`,'giu'))))"
        :key="note.uuid"
        :note="note"
        @click.native="editMode(note,index)"
      />
    </div>
    <div v-else>
      <div class="text-base font-bold text-center centered lg:text-4xl">
        <span>Loading</span>
        <span class="animated fadeIn">.</span>
        <span class="animated fadeIn delay-1s">.</span>
        <span class="animated fadeIn delay-2s">.</span>
      </div>
    </div>
    <b-modal
      :active.sync="showModal"
      trap-focus
      full-screen
      :can-cancel="false"
      aria-modal
    >
      <router-view />
    </b-modal>
  </div>
</template>

<script>
import Note from '@/components/Note/Note'
import { PlusIcon } from 'vue-feather-icons'

export default {
  name: 'Home',
  components: { Note, PlusIcon },
  data () {
    return {
      showModal: this.$route.meta.showModal
    }
  },
  watch: {
    '$route.meta' ({ showModal }) {
      this.showModal = showModal
    }
  },
  mounted () {
    if (this.$store.state.notes.length === 0) {
      this.$store.commit('setLoading', { load: 'localNotes', isLoading: true })
      this.$store.dispatch('loadLocalNotes')
    }
    // Needed after the first login
    if (navigator.onLine) {
      if (!this.$socket.connected) {
        this.$socket.connect()
      }
    } else {
      if (this.$socket.connected) this.$socket.disconnect()
      this.$store.commit('setLoading', { load: 'remoteNotes', isLoading: false })
    }
  },
  methods: {
    addNewNote () {
      this.$store.commit('setState', { name: 'selectedTag', value: null })
      this.$store.commit('showMainMenu', false)
      this.$store.dispatch('addNewNote')
    },
    editMode (note, index) {
      this.$store.commit(
        'setState',
        { name: 'top', value: document.documentElement.scrollTop || document.body.scrollTop }
      )
      this.$store.commit('setState', { name: 'selectedNote', value: note })
      this.$router.push({ path: `note/${note.uuid}` })
    }
  }
}
</script>
<style lang="sass">
.notes
  transition: 0.2s
  column-gap: 0
  column-count:1
  @apply pb-5
  @screen sm
    column-count: 2
  @screen md
    column-count: 2
  @screen lg
    column-count: 3
  @screen xl
    column-count: 4
.scrollable
  max-height: 296px
  @apply overflow-y-auto
</style>
