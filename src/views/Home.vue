<template>
  <div class="home">
   <div>
     <button @click="make()">wefwef</button>
   </div>
  </div>
</template>

<script>
import firebase from 'firebase/app'
import 'firebase/database'

export default {
  name: 'home',
  components: {
  },
  methods:{
    make(){
      firebase.database().ref('/').set({
        test:'test4'
      })
    }
  }
}
</script>
