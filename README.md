---
title: Login en Vuejs con JWT desde Express
date: "2021-05-29T10:46:37.121Z"
template: "post"
draft: false
slug: "Login-en-vuejs-con-jwt-desde-express/"
category: "Conceptos avanzados de Vuejs"
tags:
  - "Vuejs"
  - "Programación"
  - "Javascript"
  - "Frameworks"
description: "En este tutorial crearemos las vistas necesarias para cubrir las necesidades de registro y login de usuario para una plataforma web utilizando Vuejs."
socialImage: "/media/vue.jpg" 
---

La implementación de un login en el front es una de las actividades que todo desarrollador frontend debe dominar, ya que es una parte muy importante de una plataforma web. Además es una buena forma de practicar para los que están comenzando con el desarrollo ya que toca muchos temas importantes como son la seguridad por JWT, el almacenamiento en el local storage del navegador y las peticiones o request hacia un endpoint.

En este tutorial crearemos las vistas necesarias para cubrir las necesidades de registro y login de usuario para una plataforma web utilizando Vuejs.

## Índice

1. [Configuración inicial](#1-configuración-inicial)
2. [Inicio de sesión](#2-inicio-de-sesión)
3. [Ruta protegida](#3-ruta-protegida)
4. [Mostrando datos protegidos](#4-mostrando-datos-protegidos)
5. [Cerrar sesión](#5-cerrar-sesión)
6. [Registrar usuario](#6-registrar-usuario)
7. [Tu turno](#7-tu-turno)

## 1. Configuración inicial

Para este tutorial utilizaremos el [backend de autenticación de usuario que creamos en un tutorial anterior](https://andygeek.com/posts/Mis%20apuntes%20JavaScript/posts/Api-rest-de-login-con-express-mongodb-jwt/), si es que no lo leíste te recomiendo descargar el repositorio que está en dicho tutorial.

Comenzaremos creando el proyecto con `vue create` para el cual escogeremos Vue 2.0, Babel, Vuex y Vue Raouter. Luego de tener nuestro proyecto creado ya estamos listos para comenzar a colocar código.

Comenzaremos creando los campos de usuario y contraseña dentro de nuestra vista de Home. Por lo que nuestro archivo `home.vue` quedará de la siguiente manera.

```html
<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <form @submit.prevent="">
        <input type="text" placeholder="Ingrese Email" v-model="user.email">
        <input type="password" placeholder="Ingrese contraseña" v-model="user.password">
        <button type="submit">Acceder</button>
    </form>
  </div>
</template>

<script>
export default {
  name: 'Home',
  data() {
    return {
      user: {
        email: '',
        password: ''
      }
    }
  }
}
</script>
```

Lo siguiente que haremos será cambiar la vista About a Dashboard. Esta será nuestra ruta protegida por lo que utilizaremos el siguiente template.

```html
<template>
  <div class="dashboard">
    <h1>Esta es una ruta protegida</h1>
  </div>
</template>
```
También debemos actualizar el archivo de rutas `router/index.js` y `App.vue` ya que, dejaremos de utilizar el componente About y lo renombraremos como Dashboard.

## 2. Inicio de sesión

Para esto crearemos un action llamado `login` el cual mediante el método `fetch` hará una petición a nuestro API de login enviándole los datos de usuario y devolviéndonos el token si es que el usuario existe en la base de datos.

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    token: null
  },
  mutations: {
    setToken(state, payload){
      state.token = payload
    }
  },
  actions: {
    async login({ commit }, user) {
      try {
        const res = await fetch('http://localhost:8005/api/user/login', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(user)
        })
        const userDB = await res.json()
        commit('setToken', userDB.data.token)
        localStorage.setItem('token', userDB.data.token)
      } catch (error) {
        console.log('Error: ', error)
      }
    }
  },
  modules: {
  }
})
```
Explicando un poco:
- Lo primero que hacemos es crear un `state` o estado para almacenar el token.
- Después creamos un `mutation` que es el que se encargará de cambiar el token de nuestro estado, con el que traemos del endpoint.
- Y en tercer lugar creamos el `action` que nos servirá para obtener el token luego de enviar los datos del usuario al endpoint de login de nuestra API.
- En este action también utilizamos el `localStorage.setItem()` para almacenar el token que trajimos del backend.

Por otra parte dentro del archivo `home.vue` tenemos que agregar el action que activará el envió de los datos del fron hacia el back.

```html
<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <form @submit.prevent="login(user)">
        <input type="text" placeholder="Ingrese Email" v-model="user.email">
        <input type="password" placeholder="Ingrese contraseña" v-model="user.password">
        <button type="submit">Acceder</button>
    </form>
  </div>
</template>

<script>
import { mapActions } from 'vuex'

export default {
  name: 'Home',
  data() {
    return {
      user: {
        email: '',
        password: ''
      }
    }
  },
  methods: {
    ...mapActions(['login'])
  }
}
</script>
```

Con eso ya tenemos todo lo necesario para que nuestro login envíe los datos y obtengamos el token del usuario el cual es almacenado en el localStorage. Para comprobarlo, podemos utilizar las herramientas de desarrollador del navegador y buscar lo que tenemos almacenado en el localStorage. Ahi debemos encontrar el `token` y cada vez que actualicemos la página el token seguirá almacenado en el localStorage. Entonces lo siguiente es tomar el `token` del localStorage y almacenarlo nuevamente en el estado de Veux.

Para esto simplemente creamos otro action llamado `getToken`, que tomará el token y lo almacenará en el estado de Vuex.

```js
getToken({commit}) {
  if(localStorage.getItem('token')) {
    commit('setToken', localStorage.getItem('token'))
  } else {
    commit('setToken', null)
  }
}
```

Y este action lo debemos accionar al momento de que el framework cree nuestra web, por lo que utilizaremos el archivo `App.vue` donde colocaremos el action dentro del método `create()`. Y además mostraremos el token en pantalla para saber si está dentro de nuestro estado de Vuex.

```html
<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/dashboard">Dashboard</router-link>
    </div>
    <h5>Token</h5>
    <p>{{$store.state.token}}</p>
    <router-view/>
  </div>
</template>
<script>
import { mapActions } from 'vuex'
export default {
  methods: {
    ...mapActions(['getToken'])
  },
  created(){
    this.getToken()
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
```
Con esto cada vez que hagamos login en nuestra web podremos ver el token asociado a este que viene directamente del estado de Vuex. Además, si actualizamos el navegador podremos mantener el token del usuario.

Sin embargo, el token solo deberiamos verlo dentro de una ruta protegida, es decir el token debería salir dentro de nuestro dashboard.

## 3. Ruta protegida

Quitaremos el token de nuestra vista Home y la pondremos dentro de nuestra ruta protegida que en nuestro caso es el dashboard. 

```html
<template>
  <div class="about">
    <h1>Ruta protegida</h1>
    {{token}}
  </div>
</template>
<script>
import {mapState} from 'vuex'
export default {
  computed: {
    ...mapState(['token'])
  }
}
</script>
```

Lo siguiente será proteger la ruta `dashboard`, es decir debemos hacer que la ruta se habrá solo y solo si se tiene un token almacenado, si no tenemos token la ruta no se debe abrir.

Para esto trabajaremos en la ruta así que nos dirigimos al archivo `router/index.js`. Aqui primero indicaremos que la ruta dashboard requiere autenticación y lo segundo que haremos será utilizar una función que nos permita verificar si estamos pasando a una ruta protegida o no, de tal manera que hagamos la validación si es que tenemos un token en el estado de nuestra aplicación.

```js
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from '../views/Home.vue'
import Dashboard from '../views/Dashboard.vue'
import store from '../store'

Vue.use(VueRouter)

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/dashboard',
    name: 'Dashboard',
    component: Dashboard,
    // Indicamos que la ruta requiere autenticación
    meta: { requireAuth: true }
  }
]

const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes
})

// Verificando si hay token en el estado
// Este método realiza una logica al cambiar de rutas
router.beforeEach((to, from, next)=>{
  // Usamos to para verificar si requiera autenticación
  const protectedRoute = to.matched.some(record => record.meta.requireAuth)
  // Procedemos a verificar el token
  if (protectedRoute && store.state.token === null) {
    next({name: 'Home'})
  } else {
    next()
  }
})

export default router
```
Como puedes ver aquí hacemos una validación por cada cambio de ruta que suceda dentro de nuestra plataforma. Primero verificamos si la ruta a la que nos dirigimos `to` es una ruta protegida o no. Y luego verificamos si tenemos el token. Si esto no sucede nos devolvemos a la vista de Home.

## 4. Mostrando datos protegidos

Lo que haremos ahora es hacer una petición a una ruta protegida que tenemos dentro de nuetro backend, al cual enviaremos el token que tenemos, de tal manera que obtengamos los datos del usuario. Entonces crearemos un nuevo action en Vuex. A este `action` lo llamaremos `dashboard`. 

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    token: null,
    user: {
      name: ''
    }
  },
  mutations: {
    ...
    setUser(state, payload) {
      state.user.name = payload.name,
      state.user.email = payload.email
    }
  },
  actions: {
    ...
    async dashboard({commit}, auth_token){
      try {
        const res = await fetch('http://localhost:8005/api/dashboard',  {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'auth-token': auth_token
          },
        })
        const userDB = await res.json()
        commit('setUser', {name: userDB.data.user.name})
      } catch (error) {
        console.log('Error: ', error)
      }
    },
  },
  modules: {
  }
})
```
También creamos un `state` para almacenar los datos del Usuario, luego también creamos un `mutation` para cambiar los datos del usuario. Nuestro `action` toma como parámetro el token, que se lo enviaremos desde la vista, aunque también podríamos tomarlo directamente desde aqui. Este action hará GET a la ruta protegida enviando el token en el header para luego obtener de ella los datos del usuario.

## 5. Cerrar sesión

Para cerrar sesión lo que tenemos que hacer es eliminar el token que tenemos en el local storage y luego hacer un reload a la web, ya que eso eliminará automaticamente todos lo datos del estado de Vuex. Para esto comenzaremos creando un botón de Logout en la vista principal de nuestra aplicación.

```html
<button type="button" @click="logout">
  Logout
</button>
```
Luego de eso crearemos el `action` y luego el `mutation`. Esto es simple, el action llamará al mutation y este se encargará de remover el token del localStorage y hacer el reload de la web.

```js
// Action
logout ({ commit }) {
  commit('logout')
}

// Mutation
logout() {
  localStorage.removeItem('token')
  location.reload()
}
```
Podríamos hacer esto desde la misma vista, sin embargo, tener una action para esto nos sirve para reutilizarlo en cualquier parte de la aplicación.

## 6. Registrar usuario

Lo que haremos básicamente será enviar las credenciales de usuario tales como `name`, `email` y `password` mediante una petición POST al endpoint `/register` que tenemos en el API. De esta forma se almacena los datos de un nuevo usuario en la base de datos.

La vista para nuestra ruta de registrar será la siguiente. Recuerda asignar una ruta en `router/index.js` a esta vista y agregar el botón correspondiente para cambiar de ruta en tu vista principal.

```html
<template>
    <div class="register">
    <img alt="Vue logo" src="../assets/logo.png">
    <form @submit.prevent="register(user)">
        <input type="text" placeholder="Ingrese Nombre" v-model="user.name">
        <input type="text" placeholder="Ingrese Email" v-model="user.email">
        <input type="password" placeholder="Ingrese contraseña" v-model="user.password">
        <button type="submit">Registrar</button>
    </form>
  </div>
</template>
<script>
import { mapActions } from 'vuex'

export default {
    name: 'Register',
    data() {
        return {
            user:  {
                name: '',
                email: '',
                password: '',
            }
        }
    },
      methods: {
        ...mapActions(['register'])
    }
}
</script>
```
También procedemos a crear el action correspondiente al que enviamos el usuario y llamamos `register`.

```js
async register( user ) {
  try {
    const res = await fetch('http://localhost:8005/api/user/register', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },  
      body: JSON.stringify(user)
    })
    const userDB = await res.json()
    consol.log(userDB)
  } catch (error) {
    console.log('Error: ', error)
  }
},
```
Este action lo que hace es básicamente enviar las credenciales del usuario para que el back se encargue de validarlos y almacenarlos en MongoDB. El flujo siguiente debería ser el de invitar al usuario a hacer login con su cuenta de usuario nueva.

## 7. Tu turno

Con eso ya tendríamos un inicio de sesión completo en el front. Puedes implementar este flujo dentro de tus proyecto y por supuesto, mejorarlos agregando más validaciones, más seguridad y más diseño. El repositorio de este proyecto es el siguiente https://github.com/andygeek/login-vuejs.
