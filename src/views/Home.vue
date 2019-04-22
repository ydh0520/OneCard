<template>
  <div class="home">
   <div>
     <li v-for="item in roomlist" :key=item>
       <div>
        {{item.title}}
       </div>
     </li>

     <input type="text" v-model="inputtitle">
     <button @click="makeroom()">방만들기</button>
     <button @click="getRoomList()">새로고침</button>
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
      error:'',
      roomlist:[]
    }
  },
  methods:{
    makeroom(){
      if(this.inputtitle!=''){
        var room=firebase.database().ref('/room').push({
          title : this.inputtitle,
          player:['me'],
          option:{
            playerlimit:6,
            rule:'basic'
          }
        })
        this.currentroomid=room.key
        this.$router.push('/game/'+this.currentroomid)
      }
    },
    removeroom(){
      firebase.database().ref('/room/'+this.currentroomid).remove();
    },
    getRoomList(){
      var $vm=this;
      var room=firebase.database().ref('/room');
      this.roomlist=[]
      room.once('value',function(snapshot){
        var list=snapshot.val();
        snapshot.forEach(function(childSnapshot){
          $vm.roomlist.push(childSnapshot.val());
        })
      });
    }
  },
  created(){
    this.getRoomList();
  }
}
</script>
