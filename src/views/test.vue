<template>
<div class="col-12">
    <div v-if="info==''" >
        <h1>배팅 도우미</h1>
        <div class="row">
            <b-button @click="init()" variant="success" class="col-5 ml-auto mr-auto">만들기</b-button>
            <b-button @click="enter()" variant="primary" class="col-5 ml-auto mr-auto">참가</b-button>
        </div>
        <div></div>
    </div>
    <div v-if="info.ing==0">
        <h1>현재 참가자</h1>
        <hr>
        <div>  
            <li v-for="(item,index) in player" :key="index">
                player {{index}}
            </li>
        </div>
        <hr>
        <b-button v-if="playernum==0" variant="danger" @click="start()" class="col-11">시작</b-button>
        <p v-else>방장이 시작을 누를때까지 기다리세요</p>
    </div>

    <div v-if="info.ing==1">
        <h1>Player{{playernum}}</h1>
        <div>
            <li v-for="(item,index) in player" :key="index" >
                <p>player {{index}}: {{item.value}}</p>
            </li>
        </div>
        <div>
            {{game.totalprice}}
        </div>

        <div v-if="game.turn==playernum" class="row">
            <b-button @click="call()" variant="success" class="col-4 p-1">call</b-button>
            <b-button @click="double()" variant="danger" class="col-4 p-1">double</b-button>
            <b-button @click="die()" variant="dark" class="col-4 p-1">die</b-button>
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
               this.player[this.playernum].state=1;
            this.next();
            var totalprice=this.game.totalprice+this.game.betting;
            var next=this.game.turn
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
            
            this.player[this.playernum].value-=this.game.betting;
            this.player[this.playernum].state=2;
            this.next();
            var totalprice=this.game.totalprice+this.game.betting;
            var next=this.game.turn
            firebase.database().ref('/foker/player/'+this.playernum).update(this.player[this.playernum]);
            
            firebase.database().ref('/foker/game').update({
                turn:next,
                totalprice:totalprice
            });
        },
        die(){
            this.player[this.playernum].state=3;
            firebase.database().ref('/foker/player/'+this.playernum).update(this.player[this.playernum]);
            
            this.next();
            firebase.database().ref('/foker/game').update({
                turn:this.game.turn
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
            for(let i =0;i<this.info.playercounter;i++){
                this.game.turn=(this.game.turn+1)%this.info.playercounter;
                if(this.player[this.game.turn].state!=3)
                    return;
            }
        }
    }
}
</script>

<style scoped>
li {
    list-style-type: none;
}

h1{
    font-size: 3.5em
}
</style>
