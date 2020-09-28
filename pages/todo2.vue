/* eslint-disable no-console */
/* eslint-disable no-console */
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
import Vue, { PropOptions } from 'vue'

interface Todo {
  id: Number,
  name: String,
  todo: String
}

export default Vue.extend({
  props: {
    editedTodo: {
      type: Object,
      required: true
    } as PropOptions<Todo>,
    editedIndex: {
      type: Number,
      required: true
    }
  },
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
    todoList: [],
    editedIndex: -1,
    editedTodo: {
      id: 0,
      name: '', // OK
      todo: ''
    },
    defaultTodo: {
      name: '',
      todo: ''
    },

  }),

  computed: {
    formTitle () :string { // 戻り値の書き方
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
      this.editedIndex = this.todoList.indexOf(item)
      this.editedTodo = Object.assign({}, item)
      this.dialog = true
    },

    deleteTodo (id: Number) :void{
      // eslint-disable-next-line no-console
      console.log('id', id)
      const index = this.todoList.indexOf(id)
      if (confirm('Are you sure you want to delete this item?')) {
        this.todoList.splice(index, 1)
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
        this.editedTodo = Object.assign({}, this.defaultTodo)
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
