<template>
<div>
    <div>
        <button @click="init()">초기화</button>
        <button v-if="info==''" @click="enter()">참가</button>
        <div></div>
    </div>
    <div v-if="info.ing==0">
        <h3>참가자</h3>
        
        <li v-for="(item,index) in player" :key="index">
            player {{index}}
        </li>

        <button v-if="playernum==0" @click="start()">시작</button>
    </div>

    <div v-if="info.ing==1">
        <h1>Player : {{playernum}}</h1>
        <div>
            <li v-for="(item,index) in player" :key="index" >
                <p>player {{index}}: {{item.value}}</p>
            </li>
        </div>
        <div>
            {{game.totalprice}}
        </div>

        <div v-if="game.turn==playernum">
            <button @click="call()">
                call
            </button>
            <button @click="double()">
                double
            </button>
            <button @click="die()">
                die
            </button>
        </div>

        <div v-if="game.turn==game.rise">
            <button @click="accept()">내가이김</button>
        </div>
    </div>


   
</div>
</template>

<script>
import firebase from 'firebase/app'
import 'firebase/database'
export default {
    data(){
        return {
            playernum:'',
            info:'',
            game:'',
            player:''
        }
    },
    methods:{
        init(){
            var foker=firebase.database().ref('/foker');
            foker.set(
                {
                    'info':{
                        ing:0,
                        playercounter:1,
                        gamecounter:0
                    },
                    'game':{
                        turn:0,
                        rise:0,
                        totalprice:0,
                        betting:1,
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
                $vm.game=snapshotData.game;
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
                $vm.game=snapshotData.game;
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
            this.player[this.playernum].value-=this.game.betting;

            var next=(this.game.turn+1)%this.info.playercounter;
            var totalprice=this.game.totalprice+this.game.betting;
            
            firebase.database().ref('/foker/player/'+this.playernum).update(this.player[this.playernum]);
            firebase.database().ref('/foker/game').update({
                turn:next,
                totalprice:totalprice
            });
        },
        double(){
            this.game.rise=this.playernum;
            this.game.betting*=2;
            
            firebase.database().ref('/foker/game').update(this.game)

            this.call();
        },
        die(){
            this.player[this.playernum].state=1;
            firebase.database().ref('/foker/player/'+this.playernum).update(this.player[this.playernum]);
            
            var next=(this.game.turn+1)%this.info.playercounter;
            firebase.database().ref('/foker/game').update({
                turn:next
            });
        },
        accept(){
            let result = this.player[this.playernum].value+=this.game.totalprice;

            firebase.database().ref('/foker/game').update({
                betting:1,
                rise:this.playernum,
                totalprice:0,
                turn:this.playernum
            })
            this.player[this.playernum].value=result;
            
            for(let i=0;i<this.info.playercounter;i++){
                this.player[i].state=0;
            }
            
            firebase.database().ref('/foker/player').update(this.player);
        },
        next(){

            
            return 
        }
    }
}
</script>

<style scoped>
li {
    list-style-type: none;
}
</style>
