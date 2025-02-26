<template>
  <v-app-bar class="Header" hide-on-scroll flat app>
    <v-toolbar-title>{{ $route.meta.title }}</v-toolbar-title>
    <v-spacer></v-spacer>
    <v-btn icon :style="`transform: rotate(${$vuetify.theme.dark ? '18' : '0'}0deg)`" @click="setDarkMode">
      <v-icon>{{ mdiThemeLightDark }}</v-icon>
    </v-btn>
    <v-menu content-class="Header_vmenu" transition="slide-y-transition" bottom>
      <template v-slot:activator="{ on, attrs }">
        <v-btn icon v-bind="attrs" v-on="on">
          <v-icon>{{ mdiDotsVertical }}</v-icon>
        </v-btn>
      </template>
      <v-list>
        <v-subheader class="overline"><v-icon small>{{ mdiApplicationImport }}</v-icon> {{ $t('header.import') }}</v-subheader>
        <v-list-item>
          <v-list-item-title @click="importData"><v-icon>{{ mdiFileCodeOutline }}</v-icon> {{ $t('header.jsonData') }}</v-list-item-title>
        </v-list-item>
        <v-divider />
        <v-subheader class="overline"><v-icon small>{{ mdiApplicationExport }}</v-icon>  {{ $t('header.export') }}</v-subheader>
        <v-list-item>
          <v-list-item-title @click="exportData('json')"><v-icon>{{ mdiFileCodeOutline }}</v-icon>{{ $t('header.jsonData') }}</v-list-item-title>
        </v-list-item>
        <v-list-item>
          <v-list-item-title @click="exportData('s13')"><v-icon>{{ mdiFilePdf }}</v-icon>
           {{ $t('header.s13') }}</v-list-item-title>
        </v-list-item>
        <v-divider v-if="electronTopIframe" />
        <v-list-item v-if="electronTopIframe">
         <v-list-item-title @click="openFolder"><v-icon>{{ mdiFolderImage }}</v-icon>{{ $t('header.folderImages') }}</v-list-item-title>
        </v-list-item>
        <v-divider />
        <v-list-item>
          <v-list-item-title @click="dialogSetting = true"><v-icon>{{ mdiCogOutline }}</v-icon>
           {{ $t('header.settings') }}</v-list-item-title>
        </v-list-item>
      </v-list>
    </v-menu>
    <v-dialog v-model="loading" transition="dialog-top-transition" fullscreen>
      <v-overlay :value="loading">
        <v-progress-circular indeterminate size="64"></v-progress-circular>
      </v-overlay>
    </v-dialog>
    <v-dialog :value="dataToUpdate" content-class="Header__DialogDataToCheck" transition="dialog-bottom-transition" fullscreen hide-overlay>
      <v-card>
        <v-toolbar color="primary" dense dark>
          <v-btn icon dark @click="dataToCheck = []">
            <v-icon>{{ mdiClose }}</v-icon>
          </v-btn>
          <v-toolbar-title>{{ $t('header.checkChanges') }}</v-toolbar-title>
        </v-toolbar>
        <v-card-text>{{ $tcolon('header.moreRecentChanges') }}</v-card-text>
        <div v-if="dataToUpdate" class="table-check">
          <span class="h c2">{{ dataToUpdate.name }}</span>
          <span class="h">{{ $t('header.current') }}</span>
          <span class="h">{{ $t('header.imported') }}</span>
          <span>{{ $formatDate(dataToUpdate.current.updated_at, true) }}</span>
          <span>{{ $formatDate(dataToUpdate.imported.updated_at, true) }}</span>
          <template v-for="key in dataToUpdate.keys">
            <template v-if="['outAt', 'inAt', 'date'].includes(key)">
              <span :key="'a'+key">{{ $formatDate(dataToUpdate.current[key]) }}</span>
              <span :key="'b'+key">{{ $formatDate(dataToUpdate.imported[key]) }}</span>
            </template>
            <template v-else-if="['territoryId', 'peopleId'].includes(key)">
              <span :key="'a'+key">{{ getNameFromId(key, dataToUpdate.current[key]) }}</span>
              <span :key="'b'+key">{{ getNameFromId(key, dataToUpdate.imported[key]) }}</span>
            </template>
            <template v-else-if="dataToUpdate.current[key] != undefined && dataToUpdate.imported[key] != undefined">
              <span :key="'a'+key">{{ dataToUpdate.current[key] }}</span>
              <span :key="'b'+key">{{ dataToUpdate.imported[key] }}</span>
            </template>
          </template>
        </div>
        <v-card-actions>
          <v-btn class="mr-4" color="warning" text @click="dataToCheck.pop()">{{ $t('form.cancel') }}</v-btn>
          <v-btn class="mr-4" color="success" @click="validData">{{ $t('header.validateTheChange') }}</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="dialogPassword" content-class="Header__DialogPassword" max-width="600px" transition="dialog-bottom-transition">
      <v-card>
        <v-toolbar color="primary" dense dark>
          <v-toolbar-title>{{ $tcolon('password.enter') }}</v-toolbar-title>
        </v-toolbar>
        <v-card-text>
          <v-text-field v-model="aPassword" :label="$t('password.label')" />
          <template v-if="isExport">
            <p v-if="aPassword" class="green--text">{{ $t('header.fileWillBeEncrypted') }}</p>
            <p v-else class="red--text">{{ $t('header.fileWillNotBeEncrypted') }}</p>
          </template>
        </v-card-text>
        <v-card-actions>
          <v-btn class="mr-4" color="success" @click="validPassword">{{ $t('form.validate') }}</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="dialogSetting" content-class="Header__DialogSettings" max-width="600px" transition="dialog-bottom-transition">
      <v-card>
        <v-toolbar color="primary" dense dark>
          <v-toolbar-title>{{ $t('header.settings') }}</v-toolbar-title>
        </v-toolbar>
        <v-list  class="mt-4">
          <v-list-item>
            <v-list-item-icon>
              {{ $tcolon('header.language') }}
            </v-list-item-icon>
            <v-list-item-content>
              <LocaleChanger />
            </v-list-item-content>
          </v-list-item>
        </v-list>
        <v-card-actions>
          <v-btn class="mr-4" color="success" @click="dialogSetting = false">{{ $t('form.validate') }}</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-app-bar>
</template>

<script>
import { mdiClose, mdiThemeLightDark, mdiDotsVertical, mdiApplicationImport, mdiApplicationExport, mdiFilePdf, mdiCogOutline, mdiFileCodeOutline, mdiFolderImage } from '@mdi/js'
import LocaleChanger from '../components/LocaleChanger'

export default {
  name: 'Header',
  components: {
    LocaleChanger
  },
  data () {
    return {
      mdiClose,
      mdiThemeLightDark,
      mdiDotsVertical,
      mdiApplicationImport,
      mdiApplicationExport,
      mdiFilePdf,
      mdiCogOutline,
      mdiFileCodeOutline,
      mdiFolderImage,
      loading: false,
      dataToCheck: [],
      isExport: false,
      dialogPassword: false,
      dialogSetting: false,
      aPassword: '',
      electronTopIframe: false
    }
  },
  computed: {
    dataToUpdate () {
      if (!this.dataToCheck.length) { return null }
      const last = this.dataToCheck.slice(-1)[0]
      const trads = { territories: 'plural', peoples: 'peoplePlural', withdrawals: 'withdrawals', npvs: 'doNotVisitLong' }
      const id = last.data.id
      const current = this.$store.state[last.name].find(s => s.id === id) || {}
      return {
        name: $t(`territory.${trads[last.name]}`),
        keys: [...new Set([...Object.keys(current), ...Object.keys(last.data)])].filter(k => !['id', 'updated_at'].includes(k)),
        current,
        imported: last.data
      }
    }
  },
  mounted () {
    window.top.postMessage('is-top-iframe-electron', '*')
    window.onmessage = (e) => (this.electronTopIframe = e.data === 'top-iframe-is-electron')
  },
  methods: {
    importData () {
      this.loading = true
      this.$store.dispatch('import', () => {
        this.dialogPassword = true
        this.isExport = false
        return new Promise((resolve, reject) => {
          this._resolvePassord = () => {
          this.dialogPassword = false
            resolve()
          }
        })
      }).then((needUserCheck) => {
        this.loading = false
        this.aPassword = ''
        if (needUserCheck.length) {
          this.dataToCheck = needUserCheck
        }
      }).catch((err) => {
        alert(this.$t('error.occurred'))
        this.loading = false
      })
    },
    exportData (format) {
      if (format === 'json') {
        this.dialogPassword = true
        this.isExport = true
        this._resolvePassord = (password) => {
          this.$store.dispatch('export', { format, password }).then(() => {
            this.dialogPassword = false
            this.aPassword = ''
          })
        }
      } else {
        this.$store.dispatch('export', { format })
      }
    },
    setDarkMode () {
      const val = this.$vuetify.theme.dark = !this.$vuetify.theme.dark
      window.top.postMessage(`set-${val ? 'dark' : 'light'}-mode`, '*')
      this.$store.dispatch('setPref', { key: 'darkmode', val })
    },
    validData () {
      const last = this.dataToCheck.pop()
      this.$store.dispatch('updateData', { ...last, noUpdateAt: true })
    },
    getNameFromId (key, id) {
      if (key === 'territoryId') {
        return (this.$store.state.territories.find(t => t.id === id) || {}).name || '?'
      }
      if (key === 'peopleId') {
        const people = this.$store.state.peoples.find(p => p.id === id)
        return people ? `${people.firstname} ${people.lastname}` : this.$t('territory.peopleDeleted')
      }
      return '?'
    },
    validPassword () {
      if (this._resolvePassord) {
        this._resolvePassord(this.aPassword)
      }
    },
    openFolder () {
      window.top.postMessage('open-image-folder', '*')
    }
  }
}
</script>

<style lang="scss">
.Header {
  .v-toolbar__content {
    box-shadow: 0 3px 30px rgba(0, 0, 0, 0.1);
    text-decoration: none;
  }
  &.theme--light.v-app-bar.v-toolbar.v-sheet {
    background-color: #fff;
  }
  &_vmenu {
    .v-list-item {
      cursor: pointer;
      transition: .5s ease all;
      &:hover {
        background-color: #F5F5F5;
      }
    }
  }
  &__DialogDataToCheck, &__DialogPassword {
    .v-card {
      &__text {
        padding-top: 14px !important;
        padding-bottom: 0 !important;
      }
      &__actions {
        flex-wrap: wrap;
        justify-content: flex-end;
        margin-top: 10px;
      }
    }
  }
  &__DialogDataToCheck {
    .table-check {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 1px;
      margin: 12px 24px 0 24px;
      background-color: #e0e0e0;
      border: 1px solid #e0e0e0;
      border-radius: 4px;
      span {
        background-color: #fff;
        padding: 6px 4px;
        &.c2 {
          grid-column-end: span 2;
        }
        &.h {
          text-align: center;
          font-weight: bold;
        }
      }
    }
  }
}
</style>
