<template>
  <v-data-table
    :headers="headers"
    :items="todoList"
    sort-by="calories"
    class="elevation-1"
  >
    <template v-slot:top>
      <v-toolbar flat color="white">
        <v-toolbar-title>My CRUD</v-toolbar-title>
        <v-divider class="mx-4" inset vertical />
        <v-spacer />
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              color="primary"
              dark
              class="mb-2"
              v-bind="attrs"
              v-on="on"
            >
              New Item
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedTodo.name"
                      label="NAME"
                    />
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedTodo.todo"
                      label="TODO"
                    />
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer />
              <v-btn color="blue darken-1" text @click="close">
                Cancel
              </v-btn>
              <v-btn color="blue darken-1" text @click="save">
                Save
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon small class="mr-2" @click="editTodo(item)">
        mdi-pencil
      </v-icon>
      <v-icon small @click="deleteTodo(item.id)">
        mdi-delete
      </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn color="primary" @click="initialize">
        Reset
      </v-btn>
    </template>
  </v-data-table>
</template>

<script lang="ts">
import axios from 'axios'
import Vue from 'vue'

export default Vue.extend({
  data: () => ({
    dialog: false,
    headers: [
      {
        text: 'ID',
        align: 'start',
        sortable: false,
        value: 'id'
      },
      { text: 'NAME', value: 'name' },
      { text: 'TODO', value: 'todo' },
      { text: 'Actions', value: 'actions', sortable: false }
    ],
    Todo: {
      id: 0,
      name: '',
      todo: ''
    },
    todoList: [
      {
        id: 0,
        name: '',
        todo: ''
      }
    ],
    editedIndex: -1,
    editedTodo: {
      id: Number,
      name: String,
      todo: String
    },
    defaultTodo: {
      name: String,
      todo: String
    }
  }),

  computed: {
    formTitle () :string {
      return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
    }
  },

  watch: {
    dialog (val: any) {
      val || this.close()
    }
  },

  created () {
    this.initialize()
  },

  methods: {
    initialize () :void{
      axios
        .get('http://localhost:8000/todoList')
        .then(response => (this.todoList = response.data))
        // eslint-disable-next-line no-console
        .catch(error => console.log(error))
    },

    editTodo (item: any) :void{
      // console.log('editTodo item', item)
      this.editedIndex = this.todoList.indexOf(item)
      this.editedTodo = Object.assign({}, item)
      this.dialog = true
    },

    deleteTodo (id: number) :void{
      if (confirm('Are you sure you want to delete this item?')) {
        this.todoList = this.todoList.filter(todo => todo.id !== id)
        const params = { id }
        const qs = new URLSearchParams(params)
        axios
          .delete(`http://localhost:8000/todoList?${qs}`)
          // eslint-disable-next-line no-console
          .catch(error => console.log(error))
      }
    },

    close () :void{
      this.dialog = false
      this.$nextTick(() => {
        this.editedTodo.name = this.defaultTodo.name
        this.editedTodo.todo = this.defaultTodo.todo
        this.editedIndex = -1
      })
    },

    save () :void{
      if (this.editedIndex > -1) {
        Object.assign(this.todoList[this.editedIndex], this.editedTodo)
        axios
          .put('http://localhost:8000/todoList', {
            id: this.editedTodo.id,
            name: this.editedTodo.name,
            todo: this.editedTodo.todo
          })
          // eslint-disable-next-line no-console
          .catch(error => console.log(error))
      } else {
        axios
          .post('http://localhost:8000/todoList', {
            name: this.editedTodo.name,
            todo: this.editedTodo.todo
          })
          .then(() => this.initialize())
          // eslint-disable-next-line no-console
          .catch(error => console.log(error))
      }
      this.close()
    }
  }
})
</script>
