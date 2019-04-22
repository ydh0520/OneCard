<template>
<div>
    <div>
        <button @click="init()">초기화</button>
        <button v-if="info.ing==0" @click="start()">시작</button>
        <button v-if="info==''" @click="enter()">참가</button>
        <div></div>
    </div>
    <div v-if="info.ing==0">
        <li v-for="(item,index) in player" :key="index">
            player {{index}}
        </li>
    </div>
    <div v-if="info.ing==1">
        <li v-for="(item,index) in player" :key="index" >
            <p>player {{index}}: {{item}}</p>
        </li>
    </div>
    <div v-if="info.turn==playernum">
        <button @click="call()">
            call
        </button>
        <button @click="double()">
            double
        </button>
        <button @click="die()">
            die
        </button>
        <button v-if="info.rise==playernum" @click="accept()">정산하기</button>
    </div>
</div>
</template>

<script>
import firebase from 'firebase/app'
import 'firebase/database'
export default {
    data(){
        return {
            info:'',
            price:'',
            playernum:'',
            player:''
        }
    },
    watch:{
        playernum(){
            var pc=this.playernum
            firebase.database().ref('/foker/info').update({playercounter:pc});
            firebase.database().ref('/foker/player').push({
                pc:{
                    value:200,
                    state:0
                }
            });
        }
    },
    methods:{
        init(){
            var foker=firebase.database().ref('/foker');
            foker.set(
                {
                    'info':{
                        ing:0,
                        turn:0,
                        rise:0,
                        totalprice:0,
                        betting:1,
                        playercounter:1
                    }
                }
            );
            var $vm=this;
            foker.on('value',function(snapshot){
                var snapshotData=snapshot.val();
                $vm.info=snapshotData.info;
                $vm.player=snapshotData.player;
            });
        },
        enter(){
            var foker=firebase.database().ref('/foker');
            var $vm=this;
            foker.on('value',function(snapshot){
                var snapshotData=snapshot.val();
                $vm.info=snapshotData.info;
                $vm.player=snapshotData.player;
                $vm.playernum=snapshotData.playercounter+1;
            })
        },
        makeprofile(){

        },
        start(){
            firebase.database().ref('/foker/info').update({ing:1});
        },
        call(){
        },
        double(){
            
        },
        die(){
            
        },
        accept(){

        },
        next(){
            
        }
    }
}
</script>

<style scoped>

</style>
