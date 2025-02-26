<template>
  <v-dialog
    v-model="dialog"
    content-class="DialogNpv"
    max-width="600px"
    transition="dialog-bottom-transition"
    fullscreen hide-overlay
  >
    <v-card>
      <v-toolbar color="primary" dense dark>
        <v-btn icon dark @click="dialog = false">
          <v-icon>{{ mdiClose }}</v-icon>
        </v-btn>
        <v-toolbar-title v-if="data.id">{{ $t(`territory.${data.id.slice(0, 3) == 'new' ? 'create' : 'edit'}DoNotVisit`) }}</v-toolbar-title>
      </v-toolbar>
      <v-card-text>
        <v-container>
          <v-form ref="form" class="BasicForm" v-model="valid" lazy-validation>
            <DialogPickerDialog v-model="form.date" :label="$tcolon('form.date')" />
            <v-textarea v-model="form.address" :label="$tcolon('form.address')" rows="2" auto-grow />
            <div v-if="!form.planUrl" class="d-flex mt-2">
              <v-btn class="flex-grow-1" depressed @click="uploadFile"><v-icon left dark>{{ mdiCloudUpload }}</v-icon> {{ $t('territory.addPlan') }}</v-btn>
            </div>
          </v-form>
        </v-container>
      </v-card-text>
      <div v-if="!errorImage && form.planUrl" class="img-wrapper mx-auto">
        <v-img :src="$npvUrl(form.planUrl)" style="width: 100%;height: 100%; object-fit: cover;" @error="errorImage = true" />
        <v-btn fab dark color="warning" @click="uploadFile"><v-icon>{{ mdiSwapHorizontal }}</v-icon></v-btn>
      </div>
      <p v-if="uploadFileError" class="red--text text--lighten-1 mb-1 mx-auto">{{ uploadFileError }}</p>
      <v-card-actions>
        <v-btn color="error" class="mr-4" text @click="rm">{{ $t('form.delete') }}</v-btn>
        <v-spacer />
        <v-btn color="warning" text @click="close">{{ $t('form.cancel') }}</v-btn>
        <v-btn :disabled="!valid" color="success" class="mr-4" elevation="0" @click="save">{{ $t('form.save') }}</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import { mdiClose, mdiCloudUpload, mdiSwapHorizontal } from '@mdi/js'
import DialogPickerDialog from './DatePickerDialog'

export default {
  name: 'DialogNpv',
  components: {
    DialogPickerDialog
  },
  props: {
    data: {
      type: Object,
      required: true
    },
    visibility: {
      type: Boolean,
      required: true
    }
  },
  data () {
    return {
      mdiClose,
      mdiCloudUpload,
      mdiSwapHorizontal,
      form: {},
      valid: false,
      errorImage: false,
      uploadFileError: ''
    }
  },
  computed: {
    dialog: {
      get () {
        return this.visibility
      },
      set (v) {
        this.$emit('update:visibility', v)
      }
    }
  },
  mounted () {
    this.getForm()
  },
  watch: {
    visibility () {
      this.$nextTick(() => this.$refs.form.resetValidation())
      this.getForm()
    }
  },
  methods: {
    async save () {
      this.$refs.form.validate()
      await new Promise((resolve) => this.$nextTick(resolve))
      if (!this.valid) { return }
      this.$emit('update', this.form)
      this.close()
    },
    close () {
      this.dialog = false
      this.form = {}
    },
    rm () {
      this.$emit('update', { deleted: true, id: this.data.id })
      this.close()
    },
    getForm () {
      this.form = JSON.parse(JSON.stringify(this.data))
    },
    uploadFile () {
      this.uploadFileError = ''
      const planUrl = this.form.planUrl || `plan-${Math.random().toString(36).slice(2, 10).toUpperCase()}.jpg`
      const input = document.createElement('input')
      input.type = 'file'
      input.accept = '.jpg,.jpeg,.png,.bmp,.gif'
      input.onchange = e => { 
        const file = e.target.files[0]
        const formData = new FormData()
        formData.append('file', file)
        formData.append('name', planUrl)
        this.$store.dispatch('upload', formData)
          .then(() => {
            this.errorImage = true
            this.$set(this.form, 'planUrl', planUrl)
            this.$nextTick(() => (this.errorImage = false))
          })
          .catch((err) => {
            let data = err?.response?.data || {}
            if (typeof data === 'string') { data = JSON.parse(data) }
            let msg = data?.error || data?.file || data?.name
            if (Array.isArray(msg)) { msg = msg.join(', ')}
            this.uploadFileError = this.$t(`error.occurred${msg ? 'WithMsg' : ''}`, { msg })
          })
      }
      input.click()
    }
  }
}
</script>

<style lang="scss">
.DialogNpv {
  .v-card {
    display: grid;
    grid-template-rows: auto minmax(0, 1fr) auto;
    &__text {
      padding-top: 14px;
      padding-bottom: 0 !important;
    }
    &__actions {
      flex-wrap: wrap;
      justify-content: flex-end;
      margin-top: 10px;
    }
    .img-wrapper {
      position: relative;
      max-width: 900px;
      max-height: 900px;
      .v-btn {
        position: absolute;
        top: 10px;
        right: 10px;
      }
    }
  }
}
</style>
