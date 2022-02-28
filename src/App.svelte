<script>
  const radioInformation = new WebSocket("wss://listen.moe/gateway_v2");
  const fallback = new Audio("https://listen.moe/fallback");
  import TailwindCss from "./TailwindCSS.svelte";

  let heartbeatInterval;
  let paused = true;
  let loading = true;
  let currentSong = "None";
  let listeners = 0;
  let requester = "Auto";
  let lastPlayed = []
  let isPlaying =
    fallback.currentTime > 0 &&
    !fallback.paused &&
    !fallback.ended &&
    fallback.readyState > fallback.HAVE_CURRENT_DATA;

  function heartbeat(interval) {
    heartbeatInterval = setInterval(() => {
      radioInformation.send(JSON.stringify({ op: 9 }));
    }, interval);
  }

  function play() {
    console.log("Trying to play...");
    console.log(isPlaying);
    if (!isPlaying) {
      paused = false;
      fallback.play().then(() => {
        loading = false
      });
    }
  }
  function pause() {
    paused = true;
    fallback.pause();
  }

  radioInformation.onmessage = function (event) {
    const data = JSON.parse(event.data);
    if (data.op == 0) {
      heartbeat(data.d.heartbeat);
      radioInformation.send(JSON.stringify({ op: 9 }));
    }

    if (
      data.t !== "TRACK_UPDATE" &&
      data.t !== "TRACK_UPDATE_REQUEST" &&
      data.t !== "QUEUE_UPDATE" &&
      data.t !== "NOTIFICATION"
    )
      return;

    currentSong = data.d.song.title;
    requester = data.d.requester ? data.d.requester : "Automatic";
    listeners = data.d.listeners;
    lastPlayed = data.d.lastPlayed
  };
</script>
<svelte:head>
  <script src="https://kit.fontawesome.com/b04789bbb9.js" crossorigin="anonymous"></script>
</svelte:head>
<TailwindCss />
<main>
  <div class="h-screen  flex items-center bg-gray-100 justify-center">
    <div class="drop-shadow-lg p-4 bg-white rounded">
      <div
        id="connection"
        class="bg-green-500 p-1 text-white font-bold border-8 border-black text-center text-4xl"
      >
        Thanks for using Listen.moe!
      </div>

      <div class="font-bold text-2xl">
        Listen.moe - It's time to ditch other radios.
      </div>
      <div class="bg-zinc-800 text-white p-1 text-center text-xl">
        Now playing:
        <div class="italic font-bold text-center ">
          <!-- svelte-ignore a11y-distracting-elements -->
          <div class="marquee" id="currentSong">{currentSong}</div>
        </div>
        <div>
          <span id="requester" class="text-yellow-500 justify-left"
            >{requester}</span
          > Requested this vibe!
        </div>
        <div>
          <span id="vibin" class="text-yellow-500 text-right">{listeners}</span>
          Vibin' now!
        </div>
      </div>
      <div class="flex text-center text-xl">
        {#if paused}
          <div
            id="play"
            on:click={play}
            class="fa-solid fa-circle-play bg-blue-500 p-2 w-full cursor-pointer hover:bg-blue-700 transition-all duration-500  text-white"
          >
          </div>
        {:else}
          {#if loading}
          <div
            class=" bg-blue-500 p-2 w-full cursor-pointer hover:bg-blue-700 transition-all duration-500  text-white"
          >
            <i class="fa-solid fa-spinner animate-spin" />
          </div>
          {:else}
          <div
            id="pause"
            on:click={pause}
            class="fa-solid fa-circle-pause bg-red-500 hover:bg-red-700 cursor-pointer w-full transition-all duration-500 p-2 text-white "
          >
          </div>
        {/if}
        {/if}
      </div>
      <div class="">
        <div class="font-bold text-2xl">Last Songs</div>
        <div class="space-y-2">
          {#each lastPlayed as song}
          <div class="flex bg-zinc-700 text-white w-full items-center shadow-lg hover:scale-105 transition-all duration-300">
            <img src="https://www.piodeportes.com/wp-content/uploads/2016/03/hulk.jpg" class="w-16 object-cover h-16" alt="" />
            <div class="mx-3"><span class="font-bold">{song.title}</span> by <span class="italic font-thin">{song.artists[0].name}</span></div>
          </div>
        {/each}
      </div>
    </div>
  </div>
</main>

<style>
  :root {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
  }
        .marquee {
            overflow: hidden;
            position: relative;
        }
        .marquee div {
            position: absolute;
            width: 100%;
            height: 100%;
            margin: 0;
            line-height: 50px;
            text-align: center;
            -moz-transform: translateX(100%);
            -webkit-transform: translateX(100%);
            transform: translateX(100%);
            -moz-animation: scroll-left 2s linear infinite;
            -webkit-animation: scroll-left 2s linear infinite;
            animation: scroll-left 20s linear infinite;
        }
        @-moz-keyframes scroll-left {
            0% {
                -moz-transform: translateX(100%);
            }
            100% {
                -moz-transform: translateX(-100%);
            }
        }
        
        @-webkit-keyframes scroll-left {
            0% {
                -webkit-transform: translateX(100%);
            }
            100% {
                -webkit-transform: translateX(-100%);
            }
        }
        
        @keyframes scroll-left {
            0% {
                -moz-transform: translateX(100%);
                -webkit-transform: translateX(100%);
                transform: translateX(100%);
            }
            100% {
                -moz-transform: translateX(-100%);
                -webkit-transform: translateX(-100%);
                transform: translateX(-100%);
            }
        }
</style>
