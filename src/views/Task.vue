<template>
  <div class="task-view">
    <div class="flex flex-col flex-grow items-start justify-between px-4">
      <input
        type="text"
        :value="task.name"
        class="p-2 mr-2 flex-grow text-xl w-full font-bold outline-none"
        @keyup.enter="updateTaskProperty($event, 'name')"
        @change="updateTaskProperty($event, 'name')"
      />
      <textarea
        :value="task.description"
        class="relative bg-transparent px-2 border w-full mt-2 h-64 border-none leading-normal outline-none"
        @change="updateTaskProperty($event, 'description')"
      />
    </div>
  </div>
</template>

<script>
import { mapGetters } from "vuex";

export default {
  computed: {
    ...mapGetters(["getTask"]),
    task() {
      return this.getTask(this.$route.params.id);
    }
  },
  methods: {
    updateTaskProperty(e, key) {
      this.$store.commit("UPDATE_TASK", {
        task: this.task,
        key,
        value: e.target.value
      });
    }
  }
};
</script>

<style>
.task-view {
  @apply relative flex flex-row bg-white pin mx-4 m-32 mx-auto py-4 text-left rounded shadow;
  max-width: 700px;
}
</style>
