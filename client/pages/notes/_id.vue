<template>
  <v-app id="inspire">
    <v-content style="min-height: 100vh">
      <v-container
        class="my-12"
      >
        <v-row
          justify="center"
          no-gutters
          class="mx-md-6 px-2"
        >
          <v-dialog
            v-model="dialog"
            max-width="350"
          >
            <v-card>
              <v-card-title class="headline">Вы действительно хотите удалить данную запись?</v-card-title>

              <v-card-text>
                После удаления доступ к данной записи пропадет, а также вы потеряете баллы рейтинга за данную работу.
              </v-card-text>
              <v-card-text>
                Последствия необратимы.
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>

                <v-btn
                  color="grey"
                  text
                  @click="dialog = !dialog"
                >
                  Отмена
                </v-btn>

                <v-btn
                  color="red"
                  text
                  @click="confirmDeletion"
                >
                  Удалить
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-col
            :sm="10"
            :md="8"
            :lg="6"
            class="px-2"
          >
            <div class="mb-3">
              <v-text-field
                v-bind:class="{ 'd-none': !isEditing }"
                v-model="noteTitleInput"
                class="display-2 display-3 note-title"
                id="note-title"
                outlined
                placeholder="Название"
              ></v-text-field>
              <span
                v-bind:class="{ 'd-none': isEditing }"
                class="d-block display-2 display-sm-3"
              >{{ note.title }}</span>
            </div>

            <div class="mb-2">
              <v-text-field
                v-bind:class="{ 'd-none': !isEditing }"
                v-model="noteDescriptionInput"
                class="headline note-description"
                id="note-description"
                outlined
                placeholder="Описание"
              ></v-text-field>
              <span
                v-bind:class="{ 'd-none': isEditing }"
                class="headline font-weight-regular"
              >{{note.description}}</span>
            </div>
            <div class="d-none d-md-block title mb-2">
              <a class="author font-weight-regular mr-2" href="">{{ author.name }},</a>
              <span class="font-weight-light">{{ note.posted_at }}</span>
            </div>
            <div class="d-block d-md-none title mb-2">
                <span class="d-block font-weight-regular mb-2">Автор:
                  <a href="" class="author">{{ author.name }}</a>
                </span>
              <span class="d-block font-weight-light">{{ note.posted_at }}</span>
            </div>

            <div>
              <span class="title mb-2 d-block font-weight-regular">Предмет:
                <a class="subject" href="">{{subject.name}}</a>
              </span>
              <span class="title mb-2 d-block font-weight-regular">Преподаватель:
                <a class="teacher" href="">{{teacher.name}}</a>
              </span>
            </div>

            <div class="my-6">
              <v-btn v-bind:disabled="!note.link" x-large color="primary" class="d-sm-inline-block mr-1">
                <a style="color:white;text-decoration:none;" :href="note.link">Отрыть</a>
              </v-btn>
              <div v-if="!isEditing" class="d-inline-block">
                <div class="d-inline-block mr-1">
                  <v-btn v-bind:disabled="!note.link" x-large color="success" class="d-none d-sm-inline-block"
                         @click="editNote">
                    <a style="color:white;text-decoration:none;">Редактировать</a>
                  </v-btn>
                  <v-btn v-bind:disabled="!note.link" x-large color="success" class="d-sm-none my-2"
                         style="min-width: 0;" @click="editNote">
                    <a style="color:white;text-decoration:none;">
                      <v-icon dark>mdi-pencil</v-icon>
                    </a>
                  </v-btn>
                </div>
                <div class="d-inline-block mr-1">
                  <v-btn v-bind:disabled="!note.link" x-large color="error" class="d-none d-sm-inline-block"
                         @click="deleteNote">
                    <a style="color:white;text-decoration:none;">Удалить</a>
                  </v-btn>
                  <v-btn v-bind:disabled="!note.link" x-large color="error" class="d-sm-none my-2" style="min-width: 0;"
                         @click="deleteNote">
                    <a style="color:white;text-decoration:none;">
                      <v-icon dark>mdi-delete</v-icon>
                    </a>
                  </v-btn>
                </div>
              </div>
              <div v-else class="d-inline-block">
                <div class="d-inline-block mr-1">
                  <v-btn v-bind:disabled="!note.link" x-large color="success" class="d-none d-sm-inline-block mr-1"
                         @click="updateNote">
                    <a style="color:white;text-decoration:none;">Сохранить</a>
                  </v-btn>
                  <v-btn v-bind:disabled="!note.link" x-large color="success" class="d-sm-none my-2"
                         style="min-width: 0;" @click="updateNote">
                    <a style="color:white;text-decoration:none;">
                      <v-icon dark>mdi-check</v-icon>
                    </a>
                  </v-btn>
                </div>
                <div class="d-inline-block mr-1">
                  <v-btn v-bind:disabled="!note.link" x-large class="d-none d-sm-inline-block mr-1" @click="editNote">
                    <a style="color:inherit;text-decoration:none;">Отмена</a>
                  </v-btn>
                  <v-btn v-bind:disabled="!note.link" x-large class="d-sm-none my-2" style="min-width: 0;"
                         @click="editNote">
                    <a style="color:inherit;text-decoration:none;">
                      <v-icon dark>mdi-cancel</v-icon>
                    </a>
                  </v-btn>
                </div>
              </div>
            </div>
          </v-col>
        </v-row>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
  import axios from 'axios'

  export default {
    data: () => ({
      note: {
        title: "Неизвестное название",
        description: "Неизвестное описание",
        posted_at: "1 января 2000 г., 00:00:00",

      },
      author: {
        name: "Неизвестный пользователь"
      },
      category: '',
      subject: {
        name: "Неизвестный предмет"
      },
      teacher: {
        name: "Неизвестный преподаватель"
      },

      isEditing: false,

      noteTitleInput: '',
      noteDescriptionInput: '',

      dialog: false

    }),
    components: {},
    mounted() {
      // Time formatting options
      const options = {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
        timezone: 'UTC',
        hour: 'numeric',
        minute: 'numeric',
        second: 'numeric'
      };

      axios.get(`http://localhost:8080/api/v1/notes/${this.$route.params.id}`)
        .then(response => {
          console.log(response.data)
          this.note = response.data

          this.noteTitleInput = this.note.title
          this.noteDescriptionInput = this.note.description

          let timestamp = Date.parse(this.note.posted_at)
          this.note.posted_at = new Date(timestamp).toLocaleString("ru", options)

          axios.get(`http://localhost:8080/api/v1/users/${this.note.author_id}`)
            .then(response => {
              console.log(response.data)
              this.author = response.data
            })
            .catch(err => {
              console.log(err)
            })

          axios.get(`http://localhost:8080/api/v1/categories/${this.note.category_id}`)
            .then(response => {
              console.log(response.data)
              this.category = response.data

              axios.get(`http://localhost:8080/api/v1/subjects/${this.category.subject}`)
                .then(response => {
                  console.log(response.data)
                  this.subject = response.data
                })
                .catch(err => {
                  console.log(err)
                })
            })
            .catch(err => {
              console.log(err)
            })

          axios.get(`http://localhost:8080/api/v1/teachers/${this.note.teacher_id}`)
            .then(response => {
              console.log(response.data)
              this.teacher = response.data
            })
            .catch(err => {
              console.log(err)
            })
        })
        .catch(err => {
          console.log(err)
        })
    },

    validate({params}) {
      return /^\d+$/.test(params.id)
    },

    methods: {
      editNote: function () {
        this.isEditing = !this.isEditing
      },

      updateNote: function () {
        let data = new FormData()
        data.append("title", this.noteTitleInput)
        data.append("description", this.noteDescriptionInput)

        axios.post(`http://localhost:8080/api/v1/notes/${this.$route.params.id}`, data)
          .then(response => {
            this.note.title = response.data.title
            this.note.description = response.data.description
            return this.editNote()
          })
          .catch(err => {
            console.error(err)
          })
      },

      deleteNote: function () {
        this.dialog = !this.dialog
      },

      confirmDeletion: function () {
        this.dialog = !this.dialog

        // For some reason it doesn't work
        axios.delete(`http://localhost:8080/api/v1/notes/${this.$route.params.id}`)
          .then(response => {
            if (response.status == 200) {
              this.$nuxt.$router.replace({path: `/notes`})
            }
          })
          .catch(err => {
            console.error(err)
          })
      }
    }
  }
</script>

<style>
  .author, .subject, .teacher {
    text-decoration: none;
  }

  .author:hover, .subject:hover, .teacher:hover {
    text-decoration: underline;
  }

  .note-title, .note-description {
    padding: 0 !important;
    margin: 0 !important;
  }

  .note-title .v-input__slot, .note-description .v-input__slot {
    background: rgba(255, 255, 255, .5) !important;
    margin-bottom: 0;
  }

  .note-title .v-text-field__details, .note-description .v-text-field__details {
    display: none;
  }

  .note-title .v-text-field__slot input, .note-description .v-text-field__slot input {
    max-height: 100%;
  }
</style>
