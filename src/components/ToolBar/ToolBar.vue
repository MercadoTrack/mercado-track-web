<template>
  <div>
    <!-- drawer for tablet/mobile -->
    <v-navigation-drawer
      v-model="drawer"
      clipped
      disable-resize-watcher
      right
      app
    >
      <NavigationDrawerList />
    </v-navigation-drawer>

    <!-- toolbar -->
    <v-toolbar
      color="secondary"
      dark
      fixed
      app
      height="56"
      extension-height="48"
    >
      <template v-if="$vuetify.breakpoint.mdAndUp" v-slot:extension>
        <v-container class="py-0 ma-auto toolbar-container">
          <CategoriesDropdown />
          <v-btn
            flat
            active-class
            light
            color="grey darken-3"
            class="px-1 font-weight-light text-capitalize"
            to="/nosotros"
          >
            <span class="ml-2">Quiénes somos</span></v-btn
          >
          <v-btn
            flat
            active-class
            light
            color="grey darken-3"
            class="px-1 font-weight-light text-capitalize"
            target="_blank"
            href="https://cafecito.app/mercadotrack"
          >
            <span class="ml-2">Cómo ayudarnos</span></v-btn
          >
          <v-spacer></v-spacer>

          <v-menu
            v-if="isAuthenticated"
            offset-y
            left
            content-class="dropdown-menu"
            transition="slide-y-transition"
          >
            <template v-slot:activator="{ on }">
              <v-btn
                flat
                v-on="on"
                color="grey darken-3"
                class="px-3 font-weight-light text-capitalize"
                >Mi cuenta</v-btn
              >
            </template>
            <v-card>
              <MyAccountList :user="user" />
            </v-card>
          </v-menu>
          <v-btn
            v-else
            flat
            active-class
            light
            color="grey darken-3"
            class="px-1 font-weight-light text-capitalize"
            :disabled="authenticating"
            @click="login()"
            >Ingresar</v-btn
          >
        </v-container>
      </template>
      <v-container
        :class="
          `py-0 ma-auto toolbar-container ${
            $vuetify.breakpoint.smAndDown ? 'px-0' : ''
          }`
        "
      >
        <v-toolbar-title
          :class="`${$vuetify.breakpoint.smAndUp ? 'ml-3 mr-3' : 'mr-1'}`"
        >
          <router-link to="/" class="d-flex">
            <img
              v-if="$vuetify.breakpoint.smAndUp"
              height="16"
              src="../../assets/mtrack_icon.png"
              alt="Icono MercadoTrack"
            />
            <img
              v-else
              height="48"
              width="48"
              src="../../assets/mtrack_logo.png"
              alt="Icono MercadoTrack"
            />
          </router-link>
        </v-toolbar-title>
        <v-tooltip v-if="!$vuetify.breakpoint.smAndDown" bottom>
          <template v-slot:activator="{ on }">
            <v-img
              v-on="on"
              max-height="30"
              max-width="30"
              :class="`${$vuetify.breakpoint.smAndUp ? ' mr-3' : 'mr-1'}`"
              :src="countries.argentina"
            ></v-img>
          </template>
          <span>Proximamente tambien en otros paises!</span>
        </v-tooltip>
        <v-text-field
          :class="$vuetify.breakpoint.smAndUp ? 'max-width-40' : 'ml-1'"
          ref="searchInput"
          height="40"
          append-icon="search"
          label="Buscar o pegar link de ML Argentina"
          v-model="searchTerm"
          @keyup.enter="search()"
          @click:append="search()"
          solo
          clearable
          hide-details
          light
        ></v-text-field>

        <v-btn
          v-if="$vuetify.breakpoint.mdAndUp"
          to="/stats"
          color="grey darken-3"
          class="d-flex font-weight-light text-none ml-auto"
          flat
          light
          active-class
        >
          <v-icon color="primary">visibility</v-icon>
          <span class="ml-2">Sincronizacion en vivo</span>
        </v-btn>
        <template v-else>
          <aside class="ml-auto" @click.stop="drawer = !drawer">
            <v-toolbar-side-icon
              class="grey--text text--darken-3"
            ></v-toolbar-side-icon>
          </aside>
        </template>
      </v-container>
    </v-toolbar>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import CategoriesDropdown from './CategoriesDropdown'
import MyAccountList from './MyAccountList'
import NavigationDrawerList from './NavigationDrawerList'
import { isLink } from '../../utils'
import { login } from '../../utils/auth'
import * as countries from '../../assets/countries'

export default {
  components: {
    CategoriesDropdown,
    NavigationDrawerList,
    MyAccountList,
  },
  data: () => ({
    countries,
    drawer: false,
    searchTerm: '',
  }),
  computed: {
    ...mapGetters({
      user: 'auth/user',
      isAuthenticated: 'auth/isAuthenticated',
      authenticating: 'auth/authenticating',
    }),
  },
  methods: {
    login,
    search() {
      this.$refs.searchInput.blur()
      if (isLink(this.searchTerm)) {
        const [rawId] = this.searchTerm.match(/(MLA-\d+)/gi) || []
        if (!rawId) {
          if (this.searchTerm.includes('/p/ML')) {
            this.$store.commit('snackbar/nonSupportedProduct')
          } else {
            this.$store.commit('snackbar/invalidSearch')
          }
          return
        }
        const id = rawId.replace('-', '')
        this.$router.push(`/articulo/${id}`)
        this.searchTerm = ''
      } else {
        const query = { ...this.$route.query, search: this.searchTerm }
        delete query.page // to start over
        this.$router.push({ name: 'search', query })
        this.$vuetify.goTo(0, { easing: 'easeInOutCubic' }) // scrolling to top
      }
    },
  },
  watch: {
    '$route.query.search'(searchTerm) {
      this.searchTerm = searchTerm
    },
  },
  mounted() {
    this.searchTerm = this.$route.query.search || ''
  },
}
</script>

<style lang="scss" scoped>
.max-width-40 {
  max-width: 40%;
}

.v-toolbar {
  z-index: 3;
}

.v-toolbar__title {
  overflow: visible;
}

.toolbar-container {
  // fixed width container inside toolbar - https://github.com/vuetifyjs/vuetify/issues/5085
  display: flex;
  align-items: center;
  flex-grow: initial;
}
</style>
