<template>
<div class="col-12">
    <div v-if="info==''" >
        <h1>배팅 도우미</h1>
        <div class="col-10 m-auto">
            <img id="pokerchip" src="@/assets/img/pokerchip.png">
        </div>
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
            <li  v-for="(item,index) in player" :key="index">
                <div id="player" class="col-11">player {{index}}</div>
            </li>
        </div>
        <hr>
        <b-button v-if="playernum==0" variant="danger" @click="start()" class="col-11">시작</b-button>
        <p v-else>방장이 시작을 누를때까지 기다리세요</p>
    </div>

    <div v-if="info.ing==1">
        <h1>Player{{playernum}}</h1>
        <hr>
        <div>
            <table class="row col-12">
                <tr class="col-12 row">
                    <td style="width:10%"></td>
                    <td style="width:30%">이름</td>
                    <td style="width:30%">점수</td>
                    <td style="width:30%">상태</td>
                </tr>
                <tr class="col-12 row" v-for="(item,index) in player" :key="index">
                    <td style="width:10%" v-if="game.turn==index" class="turn">
                        ->
                    </td>
                    <td style="width:10%" v-else>
                    </td>
                    <td style="width:30%">player{{index}}</td>
                    <td style="width:30%">{{item.value}}</td>
                    <td style="width:30%">{{bettingState[item.state]}}</td>
                </tr>
            </table>
        </div>
        <hr>
        <div>
            <table  class="col-12">
                <tr class="col-12">
                    <td style="width:80%">현재 배팅 총액</td>
                    <td style="width:20%">배팅액</td>
                </tr>
                <tr  class="col-12">
                    <td style="width:80%">
                        <h2>
                            {{game.totalprice}}
                        </h2>
                    </td>
                    <td style="width:20%">{{game.betting}}</td>
                </tr>
            </table>
        </div>
        <div id="footer">
            <div v-if="game.turn==game.rise">
                <b-button  variant="success" class="col-11" @click="accept()">내가이김</b-button>
            </div>
                <hr>
            <div v-if="game.turn==playernum" class="row m-0">
                <b-button @click="call()" variant="primary" class="col-4 p-1">call</b-button>
                <b-button @click="double()" variant="danger" class="col-4 p-1">double</b-button>
                <b-button @click="die()" variant="dark" class="col-4 p-1">die</b-button>
            </div>
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
            bettingState:['',"Call","Double","Die","ALLIN"],
            playernum:'',
            info:'',
            game:'',
            player:''
        }
    },
    methods:{
        init(){
            var poker=firebase.database().ref('/poker');
            poker.set(
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

            poker.on('value',function(snapshot){
                var snapshotData=snapshot.val();
                $vm.info=snapshotData.info;
                $vm.game=snapshotData.game;
                $vm.player=snapshotData.player;
            });
        },
        enter(){
            var poker=firebase.database().ref('/poker');
            var $vm=this;
            poker.once('value').then(function(snapshot){
                    $vm.playernum=snapshot.val().info.playercounter;
                    $vm.makeprofile();
                }
            )
            poker.on('value',function(snapshot){
                var snapshotData=snapshot.val();
                $vm.info=snapshotData.info;
                $vm.game=snapshotData.game;
                $vm.player=snapshotData.player;
            })
        },
        makeprofile(){
            firebase.database().ref('/poker/player/'+this.playernum).set({
                value:200,
                state:0
            });
            firebase.database().ref('/poker/info').update({playercounter:this.playernum+1});
        },
        start(){
            firebase.database().ref('/poker/info').update({ing:1});
        },
        call(){
            this.player[this.playernum].value-=this.game.betting;
               this.player[this.playernum].state=1;
            this.next();
            var totalprice=this.game.totalprice+this.game.betting;
            var next=this.game.turn
            firebase.database().ref('/poker/player/'+this.playernum).update(this.player[this.playernum]);
            
            firebase.database().ref('/poker/game').update({
                turn:next,
                totalprice:totalprice
            });
        },
        double(){
            this.game.rise=this.playernum;
            this.game.betting*=2;
            
            firebase.database().ref('/poker/game').update(this.game)
            
            this.player[this.playernum].value-=this.game.betting;
            this.player[this.playernum].state=2;
            this.next();
            var totalprice=this.game.totalprice+this.game.betting;
            var next=this.game.turn
            firebase.database().ref('/poker/player/'+this.playernum).update(this.player[this.playernum]);
            
            firebase.database().ref('/poker/game').update({
                turn:next,
                totalprice:totalprice
            });
        },
        die(){
            this.player[this.playernum].state=3;
            firebase.database().ref('/poker/player/'+this.playernum).update(this.player[this.playernum]);
            
            this.next();
            if(this.game.rise==this.playernum){
                this.game.rise=this.game.turn
            }
            firebase.database().ref('/poker/game').update(this.game);
        },
        accept(){
            let result = this.player[this.playernum].value+=this.game.totalprice;

            firebase.database().ref('/poker/game').update({
                betting:1,
                rise:this.playernum,
                totalprice:0,
                turn:this.playernum
            })
            this.player[this.playernum].value=result;
            
            for(let i=0;i<this.info.playercounter;i++){
                this.player[i].state=0;
            }
            
            firebase.database().ref('/poker/player').update(this.player);
        },
        next(){
            for(let i =0;i<this.info.playercounter;i++){
                this.game.turn=(this.game.turn+1)%this.info.playercounter;
                if(this.player[this.game.turn].state<3)
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

#player{
    border: solid;
    padding:0;
    margin: auto;
    border-radius: 20px;
    margin-bottom: 5px;
    background-color: #ffbd44
}

table{
    border: 2px solid black  ;
    margin: 0;
    padding:0;
}
tr{
    margin: 0;
    padding:0;

}

td{
    border:  1px solid black ;
    margin: 0;
    padding:0;
}

.turn{
    background-color: red;
}

#footer{
    width: 100%;
    left: 0;
    position: fixed;
    bottom: 0;
    padding: 5px
}

#pokerchip{
    width:80%;
}
</style>
