<template>
    <div>
    <md-dropdown-list>
    <md-dropdown-item v-for="len in lenguajes" closing @click="changeLenguaje(len.value)">{{ len.label }}</md-dropdown-item>
</md-dropdown-list>
    <md-navbar title="VueChess" hamburger mode="hide-on-med-and-down" left nav-class="blue darken-4">
    <md-nav-item v-link="{name: 'home',activeClass: 'active'}">{{ $t("home.title") }}</md-nav-item>
    <!--
    <md-nav-item v-link="{name: 'multiplayer',activeClass: 'active'}">Multijugador</md-nav-item>
    <md-nav-item v-link="{name: 'createServer',activeClass: 'active'}">Crear servidor</md-nav-item>
-->
    <md-nav-item v-link="{name: 'game',activeClass: 'active'}">{{ $t("game.title") }}</md-nav-item>
    <md-nav-item @click="showInvitesGame()" v-if="user.user && $route.name!='game'">{{ $t("invites.title") }}</md-nav-item>
    <md-nav-item v-link="{name: 'visor',activeClass: 'active', params: board.boardParms}" v-if="user.user">{{ $t("visor.title") }}</md-nav-item>
    <md-nav-item v-link="{name: 'puzzle',activeClass: 'active'}" v-if="user.user">{{ $t("puzzles.title") }}</md-nav-item>    
    <md-nav-item @click="openModal()">{{ $t("about.title") }}</md-nav-item>
    <md-nav-item v-link="{name: 'user',activeClass: 'active'}" v-if="user.user">{{ $t("user.title") }}</md-nav-item>
    <md-nav-item v-link="{name: 'loguin',activeClass: 'active'}" v-if="!user.user">{{ $t("user.login") }}</md-nav-item>
    <md-nav-item @click="logout()" v-if="user.user">{{ $t("user.logout") }}</md-nav-item>

    <md-nav-item href="javascript:void(0)" @click="openMenu($event)">
        {{lenguajeName}}<md-icon right>arrow_drop_down</md-icon>
    </md-nav-item>
  </md-navbar>
<div>
  <md-modal id="aboutModal">
        <h4>Acerca de la App</h4>
     <md-card >
     <span slot="title" class="activator">
       Juego de ajedrez online<md-icon right>more_vert</md-icon>
     </span>
     <!-- wave effect directive -->
     <div slot="image" v-wave-effect class="waves-effect waves-block waves-light">
      <img style=" width: 30%;left: 33%;" class="activator" src="./assets/2d.png">
    </div>
    <p>
      <a href="#!">Leer más</a>
    </p>
    <div slot="reveal">
      <span class="card-title grey-text text-darken-4">
        Juego de ajedrez online
      </span>
      <p>
        Juego de ajedrez multijugador online que te permitirá enfrentarte a otros jugadores en tiempo real y al propio computador. Resuelve los puzzles y crea otros nuevos.

        <h5 class="flat-text-header hello">Tecnologías</h5>
        <p>
          Vuejs, Nodejs, SocketIO, MongoDb y Webpack
        </p>

        <h5 class="flat-text-header hello">Desarrollador</h5>
        <p>
          Gustavo Crespo Sánchez gustayocs@gmail.com 2016
        </p>

      </p>
    </div>
  </md-card>
    </md-modal>
      <user-invites></user-invites>
</div>
    
  <div class="row">
      <router-view class="animated" transition="entern"></router-view>
  </div>
      </div>
   	  
</template>

<script>
import store from './vuex/store'
import UserService from './services/user'
import Board from './services/board'
import UserInvites from './components/home/invites'
import Lenguajes from './services/lenguajes'
import Store from './services/lstorage'
export default {
  data () {
    return {
      closeResult: '',
      user: UserService,
      board: Board,
      lenguajeName: 'Español',
      lenguaje: Store.get('lenguaje', 'es'),
      lenguajes: Lenguajes.lenguajes
    }
  },
  methods: {
    getLenguajeName () {
      for (var i in this.lenguajes) {
        if (this.lenguajes[i].value === this.lenguaje) {
          this.lenguajeName = this.lenguajes[i].label
        }
      }
    },
    changeLenguaje (len) {
      if (this.lenguaje !== len) {
        Store.set('lenguaje', len)
        window.location.reload()
      }
    },
    openMenu (event) {
      this.$broadcast('dropdown-list::open', event)
    },
    openModal () {
      this.$broadcast('modal::open', 'aboutModal')
    },
    showInvitesGame () {
      this.$broadcast('modal::open', 'invitesGame')
    },
    logout () {
      UserService.logout()
    },
    userLoguinSocket () {
      if (this.user.user_acces()) {
        const data = {
          c: 'chat',
          f: 'username',
          data: {
            user: UserService.getUser()
          }
        }
        setTimeout(function () {
          this.$socket.emit('event', data, function (...callbacks) {
            console.log(callbacks[1])
          })
        }.bind(this), 100)
      }
    }
  },
  events: {
    userLoguin (data) {
      this.userLoguinSocket()
    }
  },
  sockets: {
    event (data) {
      this.$broadcast(data.event, data.data)
    }
  },
  created () {
    this.userLoguinSocket()
    this.getLenguajeName()
  },
  components: {
    UserInvites
  },
  store
}
</script>

