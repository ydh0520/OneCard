<template>
  <div class="home">
   <div>
     <input type="text" :v-bind="inputtitle">
     <button @click="makeroom()">방만들기</button>
     <button @click="removeroom()">방삭제</button>
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
  data(){
    return{
      inputtitle:'',
      currentroomid:'',
    }
  },
  methods:{
    makeroom(){
      var room=firebase.database().ref('/room').push({
        title : '',
        player:['me'],
        option:{
          playerlimit:6,
          rule:'basic'
        }
      })
      this.currentroomid=room.key;
      this.$router.push('/game/'+this.currentroomid)
    },
    removeroom(){
      firebase.database().ref('/room/'+this.currentroomid).remove();
    }
  }
}
</script>
