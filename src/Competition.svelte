  <script>
    import { onMount } from 'svelte'
    import { push } from 'svelte-spa-router'
    import { backendUrl, centaurToken } from "./api";
    import Table from './Table.svelte'
    import Game from './Game.svelte'
    import Board from './Board.svelte'
    import { stepNumber } from "./storage";
    import { blackScore } from "./storage";
    import { whiteScore } from "./storage";
    import { blackStonesCount } from "./storage";
    import { whiteStonesCount } from "./storage";
    import { gameHistory } from "./storage";
    import { updateTime, elapsed } from './storage';
    import { w3cwebsocket as W3CWebSocket } from "websocket"

    let gameId = localStorage.getItem('gameId');
    let client = new W3CWebSocket('ws://172.104.137.176:41239');
    let token = localStorage.getItem('token');
    let hints = [];
    async function getHintMoves(count) {
      const response = await fetch(backendUrl+'hints/best-moves?game_id='+gameId+'&centaur_token='+
          centaurToken+'&token='+token+'&count='+count);
        const json = await response.json();
        hints = json.hint;
    }

    onMount(async () => {
      if (token === null) {
        push('/');
      }
      const response = await fetch(backendUrl + 'game/create/bot?token='+localStorage.getItem('token') , {method: 'POST'});
        const json = await response.json();
        gameId = json.gameId;
        localStorage.setItem('gameId', json.gameId);
    });
    // onMount(async () => {
    //   const response = await fetch(backendUrl + 'game/create/bot?token='+localStorage.getItem('token') , {method: 'POST'});
    //   const json = await response.json();
    //   gameId = json.gameId;
    //   localStorage.setItem('gameId', json.gameId);
    //   client.onopen = function () {
    //     client.send(JSON.stringify([5, 'go/game']));
    //     console.log(token);
    //     console.log(gameId);
    //     client.send(JSON.stringify([
    //       7, // 7 - статус: отправка сообщения
    //       "go/game", // в какой топик отправляется сообщение
    //       {
    //         command: "auth",  // команда на авторизацию подключения
    //         token: token, // токен игрока
    //         game_id: gameId // номер игры
    //       }
    //     ]));
    //     client.send(JSON.stringify([
    //       7,// 7 - статус: отправка сообщения
    //       "go/game", // в какой топик отправляется сообщение
    //       {
    //         command: "move", // команда на отправку хода
    //         token: token,  // токен игрока
    //         place: 'd13',  // место куда сделать ход, формат: d13
    //         game_id: gameId // номер игры
    //       }
    //     ]));
    //   }
    // });
    //
    // client.onmessage = function(response)  {
    //   console.log(response.data);
    // }

    function funcForTime(){
      updateTime();
    }

    $: gameState = $stepNumber%2;
    $: colorAttack = gameState ? "white" : "black";
    function funcSkip(){
      stepNumber.decrement();
      updateTime();
    }

  </script>
<svelte:head>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-eOJMYsd53ii+scO/bJGFsiCZc+5NDVN2yr8+0RDqr0Ql0h+rP48ckxlpbzKgwra6" crossorigin="anonymous">
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta3/dist/js/bootstrap.bundle.min.js" integrity="sha384-JEW9xMcG8R+pH31jmWH6WWP0WintQrMb4s7ZOdauHnUtxwoG2vI5DkLtS3qm9Ekf" crossorigin="anonymous"></script>
</svelte:head>


<div class="container-fluid">
  <div class="row ">
    <div class="col-md-3 text-align-middle">
      <div class="podskazki ">
        <h1 class="text-align-center">Подсказки</h1>
        <div class="btn-group-vertical ">
          <button class="btn" on:click={()=> getHintMoves(1)}> Получить количество ходов - 1 </button>
          <button class="btn" on:click={()=> getHintMoves(2)}> Получить количество ходов - 2 </button>
          <button class="btn" on:click={()=> getHintMoves(3)}> Получить количество ходов - 3 </button>

        </div>
      </div>

    </div>
    <div class="col-md-6">
      <div class="game">
          <Game {funcForTime} bind:hints/>
      </div>
    </div>
    <div class="col-md-3">
      <div class="row">
        <div class="players">
          <div class="player row">
            <h1>Игрок 1 - black</h1>
          </div>
          <p>Очков : {$blackScore}</p>
          <p>Камней : {$blackStonesCount}, max: 84</p>

          <div class="player row">
            <h1> Игрок 2 - white</h1>
            <p>Очков : {$whiteScore}</p>
            <p>Камней : {$whiteStonesCount}, max: 84</p>
          </div>
        </div>
      </div>
      <div class="row">
        <div class="stat">
          <div class="row ">
            <h1>История ходов</h1>
            {#each $gameHistory as element }
              <p>{element[0]}: x-{element[1]}, y-{element[2]}.</p>
            {/each}
          </div>
        </div>
      </div>
      <div class="row d-flex flex-colum align-text-bottom panel">
        <div class="col-6">
          {#if $elapsed > 59}
          <span><h2>Время: {Math.floor($elapsed/60)}:{$elapsed%60}</h2></span>
          {:else}
            <span><h2>Время: 0:{$elapsed}</h2></span>
          {/if}
          <h2>Ход:{colorAttack}</h2>
        </div>
        <div class="col-6">
          <button on:click={funcSkip}><h2>Пропуск хода</h2></button>
        </div>
      </div>
    </div>
  </div>
</div>

<style>
  body {
    margin: 1em 1em;
  }
  .container {
    margin-top: 25px;
  }
  *, .btn {
    color : #728ab7;
  }
  h1 {
    font-size: 1.7em;
    font-weight: bold;
  }
  h2, p{
    font-size: 1.4em;
    margin-bottom: 5px;
  }
  button {
    margin-bottom: 10px;
    font-size: 1.3em;
    border: 0px;
    border-radius: 10px;
  }
  .game {
    /* margin-left: -100px;
    margin-top: -50px; */
    /* display: flex;
    flex-direction: column;
    align-items: center; */
    
    /* margin-left: 40px;
    width: 850px; */

  }
  .field{
    margin-left: auto;
    margin-right: auto;
    /* height: 00px;  */
    width: 600px; 
  }

  .stat,.players {
    margin: 0 0 1em 0;
  }
  .stat, .players,.podskazki,.field{

    box-shadow: -10px -10px 30px #FFFFFF, 10px 10px 30px rgba(174, 174, 192, 0.4);
    border-radius: 16px;
    padding: 10px;
  }
  .stat{
    overflow-y: scroll;
    height: 300px;
  }

  .row h2 {
    font-size: 25px;
  }

  .row h3 {
    font-size: 15px;
  }

  .col-6 button {
    padding: 10px;
    font-size: 16px;
  }

</style>