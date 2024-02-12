<template>
  <v-footer dark padless app inset>
    <v-btn
        style='position:absolute;bottom: 50px;right: 10px;background-color: white;color:#1A535C;'
        @click='openInstructions'
    >
      ИНСТРУКЦИИ ПОРТАЛА
    </v-btn>
    <v-dialog
        v-model='instructions'
        max-width='800'
    >
      <v-card
          class='mx-auto'
          tile
      >
        <v-card-text style='text-align:center;padding: 24px 20px;'>
          <h2>ИНСТРУКЦИИ ПОРТАЛА</h2>

          <v-list>
            <v-list-item-group
                color='primary'
            >
              <v-list-item
                  v-for='(item, i) in instructionList'
                  :key='i'
                  :href='item.file'
                  target='_blank'
              >
                <v-list-item-icon style="margin-right: 10px;">
                  <v-icon>mdi-file</v-icon>
                </v-list-item-icon>
                <v-list-item-content>
                  <v-list-item-title
                      style='text-align: left'
                      v-text='item.name'
                  ></v-list-item-title>
                </v-list-item-content>
              </v-list-item>
            </v-list-item-group>
          </v-list>
        </v-card-text>
        <v-card-actions class='justify-end'>
          <v-btn
              text
              @click='instructions = false'
          >Закрыть
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-btn
        style='position:absolute;bottom: 95px;right: 10px; -webkit-text-fill-color: white; background-color: #1A535C; font-size: small'
        @click='connectUs = true'
    >
      Обратная связь с поддержкой
    </v-btn>
    <v-dialog
        v-model='connectUs'
        max-width='600'
    >
      <v-card
      >
        <v-card-text style='text-align:center;padding: 24px 20px;color: black'>
          <h2 style='margin:5px 5px 20px 5px;'>Здесь Вы можете отправить свой вопрос<br>менеджеру компании ISKER</h2>
          <v-form
              ref='form'
              v-model='valid'
              @submit.prevent="sendConnectUs"
          >
            <h3 style='margin:5px 10px; text-align: left'>
              Ваш e-mail:
            </h3>
            <v-text-field
                v-model='guest_email'
                :rules='emailRules'
                style='margin: 0 10px;'
                label='Email'
                placeholder='Укажите почту для связи с Вами'
                type='email'
                autocomplete='email'
                autofocus
                outlined
                dense
            ></v-text-field>
            <h3 style='margin:0 10px 10px 10px; text-align: left'>
              ФИО:
            </h3>
            <v-text-field
                v-model='guest_names'
                :rules='subjectRules'
                style='margin: 0 10px'
                label='ФИО'
                placeholder='Укажите Ваши ФИО'
                type='text'
                outlined
                dense
            ></v-text-field>
            <h3 style='margin:0 10px 10px 10px; text-align: left'>
              Ваш номер телефона:
            </h3>
            <v-text-field
                v-model='guest_phone'
                :rules='subjectRules'
                style='margin: 0 10px'
                label='Телефон'
                placeholder='Укажите Ваш номер телефона для связи'
                type='text'
                outlined
                dense
            ></v-text-field>

            <h3 style='margin:0 10px 10px 10px; text-align: left'>
              Тема Вашего обращения:
            </h3>
            <v-text-field
                v-model='guest_subject'
                :rules='subjectRules'
                style='margin: 0 10px'
                label='Тема письма'
                placeholder='Напишите здесь тему Вашего письма'
                type='text'
                outlined
                dense
            ></v-text-field>

            <h3 style='margin:0 10px 10px 10px; text-align: left'>
              Текст Вашего обращения:
            </h3>
            <v-textarea
                v-model='guest_text'
                :rules='textRules'
                style='margin: 0 10px'
                label='Текст письма'
                placeholder='Напишите здесь текст Вашего письма'
                type='text'
                outlined
                auto-grow
                rows='4'
            ></v-textarea>
          <v-row
              v-for='(input, key) in inputsAddFile'
              :key='key'
              style='margin: 0;width: 100%'
          >
            <v-file-input
                type="file"
                ref="fileInput"
                accept='image/*,.pdf,.xlsx,.xls, .doc, .docx'
                :rules='fileRules'
                :label='input.label'
                @change="handleFileChange($event, input, key)"
                clear-icon="false"
            >
              <template #selection='{ index, text }'>
                <v-chip
                    v-if='index < 3'
                    color='primary'
                    dark
                    label
                    small
                >
                  {{ text }}
                </v-chip>
              </template>
            </v-file-input>
          </v-row>
          <v-col>
            <v-btn
                @click='pushInput'
                style="margin-bottom: 20px"
            >
              <v-icon>mdi-plus-circle</v-icon>
              &nbsp; Добавить еще файл
            </v-btn>
            <v-btn
                v-if='this.inp > 0'
                class='error'
                style='margin:0 0 20px 0;'
                @click='deleteInput()'
            >
              <v-icon>mdi-delete</v-icon>
              Удалить последний файл
            </v-btn>
          </v-col>
          <v-btn
              width='150px'
              style='margin-right: 135px; background-color: #E43945'
              title='Отменить'
              class='refusal white--text'
              @click='connectUs = false'
          >
            <v-icon>mdi-close</v-icon>
            &nbsp;Отмена
          </v-btn>
          <v-btn
              width='250px'
              class='confirmation white--text'
              style='background-color: #17AA00'
              type='submit'
          >
            Отправить
            <v-icon>mdi-check</v-icon>
          </v-btn>
          <v-snackbar
              v-model='snack'
              :timeout='3000'
              :color='snackColor'
          >
            {{ snackText }}
            <template #action='{ attrs }'>
              <v-btn
                  v-bind='attrs'
                  class='white--text'
                  text
                  @click='snack = false'
              >
                Закрыть
              </v-btn>
            </template>
          </v-snackbar>
          </v-form>
        </v-card-text>
      </v-card>
    </v-dialog>
    <v-snackbar
        v-model='snack'
        :timeout='3000'
        :color='snackColor'
    >
      {{ snackText }}
      <template #action='{ attrs }'>
        <v-btn
            v-bind='attrs'
            class='white--text'
            text
            @click='snack = false'
        >
          Закрыть
        </v-btn>
      </template>
    </v-snackbar>
    <v-card class="flex" flat tile style='background-color: #f7fff7ff' :elevation="10">
      <v-card-text class="py-2 text-center" style='color: #1a535cff'>
        {{ new Date().getFullYear() }} <strong>©ISKER Group</strong>
      </v-card-text>
    </v-card>
  </v-footer>
</template>

<script>
export default {
  name: "Footer",
  data() {
    return {
      instructions: false, // окно с инструкциями
      instructionList: [],
      valid: false,
      connectUs: false,
      guest_email: '',
      guest_subject: '',
      guest_text: '',
      file: [],
      guest_names: '',
      guest_phone: '',
      countoffiles: 0,
      emailRules: [
        v => !!v || 'Введите Вашу почту',
        v => /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(v) || 'Вы неправильно ввели почту',
      ],
      subjectRules: [
        v => !!v || 'Это поле не может быть пустым',
        v => v && v.length <= 150 || "Максимум 150 символов",
      ],
      textRules: [
        v => !!v || 'Это поле не может быть пустым',
        v => v && v.length <= 1000 || "Максимум 1000 символов",
      ],
      fileRules: [
        value => !value || value.size < 2000000 || 'Файл превышает максимальный размер(2МБ), письмо не будет отправлено'
      ],
      snack: false,
      snackColor: '',
      snackText: '',
      formData: '',
      inputsAddFile: [
        {label: 'Прикрепить файл (Не более 2МБ)'}
      ],
      ch: 0,
      first: false,
      second: false,
      third: false,
      fourth: false,
      fifth: false,
      inp: 0,
    }
  },
  methods: {
    handleFileChange($event, input, key) {
      if (key === 0 && this.first === false) {
        this.file.push($event);
        this.first = true
      } else if (key === 0 && this.first === true) {
        this.file.splice(key, 1)
        this.file.push($event);
      } else if (key === 1 && this.second === false) {
        this.file.push($event);
        this.second = true
      } else if (key === 1 && this.second === true) {
        this.file.splice(key, 1)
        this.file.push($event);
      } else if (key === 2 && this.third === false) {
        this.file.push($event);
        this.third = true
      } else if (key === 2 && this.third === true) {
        this.file.splice(key, 1)
        this.file.push($event);
      } else if (key === 3 && this.fourth === false) {
        this.file.push($event);
        this.fourth = true
      } else if (key === 3 && this.fourth === true) {
        this.file.splice(key, 1)
        this.file.push($event);
      } else if (key === 4 && this.fifth === false) {
        this.file.push($event);
        this.fifth = true
      } else if (key === 4 && this.fifth === true) {
        this.file.splice(key, 1)
        this.file.push($event);
      }
      console.log(this.file)
      // console.log(this.file)
      // console.log(this.inputsAddFile)
    },
    // Добавить еще инпут для файла
    pushInput() {
      if (this.countoffiles < 4) {
        this.inputsAddFile.push({
          label: 'Прикрепить файл (Не более 2МБ)'
        })
        this.countoffiles += 1
        this.inp += 1
      } else {
        this.snack = true
        this.snackColor = 'error white--text'
        this.snackText = 'Можно прикрепить только 5 файлов'
      }
    },
    // Удалить инпут с файлом
    deleteInput() {
      if (this.inp > 0) {
        // console.log('selected: ' + this.inputsAddFile[key].label)
        // this.inputsAddFile.splice(key, 1)
        // this.file.splice(key, 1)
        this.inputsAddFile.pop()
        this.file.pop()
        // console.log('deleted: ' + this.inputsAddFile)
        this.countoffiles -= 1
      }
    },
    async sendConnectUs() {
      if (this.$refs.form.validate()) {
        if(!this.file.includes(undefined)) {
          for (let i = 0; i < this.file.length; i++) {
            if (this.file[i].size > 2 * 1024 * 1024) {
              this.file.splice(i, 1)
            }
          }
        }else{
        try {
          this.formData = new FormData();
          this.formData.append('email', this.guest_email);
          this.formData.append('subject', this.guest_subject);
          this.formData.append('text', this.guest_text);
          this.formData.append('names', this.guest_names);
          this.formData.append('phone', this.guest_phone);
          for (let i = 0; i < this.file.length; i++) {
            this.formData.append('files[]', this.file[i]);
          }
          await this.$axios.$post('/api/v1/contact_us/', this.formData)
          this.connectUs = false
          this.snack = true
          this.snackColor = 'success black--text'
          this.snackText = 'Ваше письмо отправлено'
          this.countoffiles = 0
          this.formData = ''
          this.inputsAddFile = [{label: 'Прикрепить файл (Не более 2МБ) '}]
          this.file.length = 0
          this.inp = 0
          this.ch = 0
          this.first = false
          this.second = false
          this.third = false
          this.fourth = false
          this.fifth = false
          this.$refs.form.reset()
        } catch (e) {
          this.connectUs = false
          console.log(e)
          this.snack = true
          this.snackColor = 'error white--text'
          this.snackText = 'Ошибка! Письмо не отправлено'
          this.countoffiles = 0
          this.formData = ''
          this.inputsAddFile = [{label: 'Прикрепить файл (Не более 2МБ) '}]
          this.file = []
          this.ch = 0
          this.inp = 0
          this.first = false
          this.second = false
          this.third = false
          this.fourth = false
          this.fifth = false
          this.$refs.form.reset()
        }
        }
      } else {
        this.snack = true
        this.snackColor = 'refusal white--text'
        this.snackText = 'Письмо не отправлено'
      }
    },
    async openInstructions() {
      const itemList = await this.$axios.$get('/api/v1/instructions/')
      if (itemList) {
        this.instructionList = itemList
        this.instructions = true
      }
    },
  }
}
</script>

<style scoped>

</style>
