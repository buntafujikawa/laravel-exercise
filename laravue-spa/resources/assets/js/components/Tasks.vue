<template>
  <div>
    <div v-if="userState.authenticated">
      <strong>Hello, {{ userState.user.name }}!</strong>
      <p>Your tasks here.</p>

      <ul v-for="task in tasks">
        <li v-if="task.is_done">
          <strike> {{ task.name }} </strike>
        </li>
        <li v-else>
          {{ task.name }}
        </li>
        <button @click="completeTask(task)" class="btn btn-sm btn-success" v-if="task.is_done">Undo</button>
        <button @click="completeTask(task)" class="btn btn-sm btn-success" v-else>Done</button>

        <button @click="removeTask(task)" class="btn btn-sm btn-danger">Remove</button>
      </ul>

      <div class="form-group">
        <div class="alert alert-danger" role="alert" v-if="showAlert">
          {{ alertMessage }}
        </div>
        <input type="text" class="form-control"
               v-model="name" @keyup.enter="addTask" placeholder="new task...">
        <button class="btn btn-primary" disabled="disabled" v-if="name === ''">
          Add task
        </button>
        <button class="btn btn-primary" @click='addTask' v-else>
          Add task
        </button>
      </div>
    </div>

    <p v-else>
      please <router-link to="/login">Login.</router-link>
    </p>

  </div>
</template>

<script>
  import http from '../services/http'
  import userStore from '../stores/userStore'
  export default {
    mounted() {
      this.fetchTasks()
    },
    data() {
      return {
        userState: userStore.state,
        tasks: [],
        name: '',
        showAlert: false,
        alertMessage: '',
      }
    },
    methods: {
      fetchTasks () {
        // TODO: not to send request when the user is not authenticated
        http.get('tasks', res => {
          this.tasks = res.data
        })
      },
      addTask () {
        if (this.name === '') {
          this.showAlert = true
          this.alertMessage = 'Task name should not be blank.'
          return false
        }
        http.post('tasks', {name: this.name}, res => {
          this.tasks[res.data.id] = res.data
          this.name = ''
          this.showAlert = false
          this.alertMessage = ''
        })
      },
      completeTask (task) {
        http.put('tasks/' + task.id, {is_done: !task.is_done}, res => {
          this.tasks[task.id] = res.data
          this.$forceUpdate()
        })
      },
      removeTask (task) {
        http.delete('tasks/' + task.id, {}, () => {
          delete this.tasks[task.id]
          this.$forceUpdate()
        })
      },
    }
  }
</script>
