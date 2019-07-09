<template>
  <div class="board">
    <div class="flex flex-row items-start">
      <div
        class="column"
        @drop="moveTaskOrColumn($event, column.tasks, $columnIndex)"
        @dragover.prevent
        @dragenter.prevent
        draggable
        @dragstart.self="pickupColumn($event, $columnIndex)"
        v-for="(column, $columnIndex) in board.columns"
        :key="$columnIndex"
      >
        <div class="flex items-center mb-2 font-bold">{{ column.name }}</div>
        <div class="list-reset">
          <div
            class="task"
            @click="goToTask(task)"
            draggable
            @dragover.prevent
            @dragenter.prevent
            @drop.stop="moveTaskOrColumn($event, column.tasks, $columnIndex, $taskindex)"
            @dragstart="pickupTask($event, $taskindex, $columnIndex)"
            v-for="(task, $taskindex) in column.tasks"
            :key="$taskindex"
          >
            <span class="w-full flex-no-shrink font-bold">{{ task.name }}</span>
            <p
              v-if="task.description"
              class="w-full flex-no-shrink mt-1 text-sm"
            >{{ task.description }}</p>
          </div>
          <input
            type="text"
            class="block p-2 w-full bg-transparent outline-none"
            placeholder="+ Enter new task"
            @keyup.enter="createTask($event, column.tasks)"
          />
        </div>
      </div>
    </div>

    <div v-if="isTaskOpen" @click.self="close" class="task-bg">
      <router-view />
    </div>
  </div>
</template>

<script>
import { mapState, mapMutations } from "vuex";
export default {
  computed: {
    ...mapState(["board"]),
    isTaskOpen() {
      return this.$route.name === "task";
    }
  },
  methods: {
    goToTask(task) {
      this.$router.push({
        name: "task",
        params: {
          id: task.id
        }
      });
    },
    close() {
      this.$router.push({
        name: "board"
      });
    },
    createTask(e, tasks) {
      this.$store.commit("CREATE_TASK", {
        tasks,
        name: e.target.value
      });
      e.target.value = "";
    },
    pickupTask(e, taskIndex, fromColumnIndex) {
      e.dataTransfer.effectAllowed = "move";
      e.dataTransfer.dropEffect = "move";

      e.dataTransfer.setData("from-task-index", taskIndex);
      e.dataTransfer.setData("from-column-index", fromColumnIndex);
      e.dataTransfer.setData("type", "task");
    },
    pickupColumn(e, fromColumnIndex) {
      e.dataTransfer.effectAllowed = "move";
      e.dataTransfer.dropEffect = "move";

      e.dataTransfer.setData("from-column-index", fromColumnIndex);
      e.dataTransfer.setData("type", "column");
    },
    moveTaskOrColumn(e, toTasks, toColumnIndex, toTaskIndex) {
      const type = e.dataTransfer.getData("type");
      if (type === "task") {
        this.moveTask(
          e,
          toTasks,
          toTaskIndex !== undefined ? toTaskIndex : toTasks.length
        );
      } else {
        this.moveColumn(e, toColumnIndex);
      }
    },
    moveColumn(e, toColumnIndex) {
      const fromColumnIndex = e.dataTransfer.getData("from-column-index");

      this.$store.commit("MOVE_COLUMN", {
        fromColumnIndex,
        toColumnIndex
      });
    },
    moveTask(e, toTasks, toTaskIndex) {
      const fromColumnIndex = e.dataTransfer.getData("from-column-index");
      const fromTasks = this.board.columns[fromColumnIndex].tasks;
      const fromTaskIndex = e.dataTransfer.getData("from-task-index");

      this.$store.commit("MOVE_TASK", {
        toTasks,
        fromTaskIndex,
        fromTasks,
        toTaskIndex
      });
    }
  }
};
</script>

<style lang="css">
.task {
  @apply flex items-center flex-wrap shadow mb-2 py-2 px-2 rounded bg-white text-grey-darkest no-underline;
}

.column {
  @apply bg-grey-light p-2 mr-4 text-left shadow rounded;
  min-width: 250px;
}

.board {
  @apply p-4 bg-teal-dark h-full overflow-auto;
}

.task-bg {
  @apply pin absolute;
  background: rgba(0, 0, 0, 0.5);
}
</style>
