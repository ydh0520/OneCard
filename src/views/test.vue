<template>
<div>
    <div>
        <button @click="init()">초기화</button>
        <button v-if="info.ing==0&&playernum==0" @click="start()">시작</button>
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
            player:'',
            myValue:200
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
                    },
                    'player':{
                        0:{
                            value:200,
                            state:0
                        }
                    }
                }
            );
            var $vm=this;
            this.playernum=0;
            foker.on('value',function(snapshot){
                var snapshotData=snapshot.val();
                $vm.info=snapshotData.info;
                $vm.player=snapshotData.player;
            });
        },
        enter(){
            var foker=firebase.database().ref('/foker');
            var $vm=this;
            foker.once('value').then(function(snapshot){
                    $vm.playernum=snapshot.val().info.playercounter;
                    $vm.makeprofile();
                }
            )
            foker.on('value',function(snapshot){
                var snapshotData=snapshot.val();
                $vm.info=snapshotData.info;
                $vm.player=snapshotData.player;
            })
        },
        makeprofile(){
            firebase.database().ref('/foker/player/'+this.playernum).set({
                value:200,
                state:0
            });
            firebase.database().ref('/foker/info').update({playercounter:this.playernum+1});
        },
        start(){
            firebase.database().ref('/foker/info').update({ing:1});
        },
        call(){
            this.myValue-=this.info.betting;
            this.totalprice+=this.info.betting;
            firebase.database().ref('/foker/player/'+this.playernum).update({
                state:0,
                value:this.myValue
            })
            this.next();
        },
        double(){
            var nextrise=this.playernum;
            var nextbetting = this.info.betting*=2;

            firebase.database().ref('/foker/info').update({rise:nextrise});
            firebase.database().ref('/foker/info').update({betting:nextbetting});
            
            this.call();
        },
        die(){
            
        },
        accept(){

        },
        next(){
            let next = (this.info.turn+1)%this.info.playercounter;
            firebase.database().ref('/foker/info').update({turn:next});
        }
    }
}
</script>

<style scoped>
li {
    list-style-type: none;
}
</style>
