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
  };
</script>
<svelte:head>
  <script src="https://kit.fontawesome.com/b04789bbb9.js" crossorigin="anonymous"></script>
</svelte:head>
<TailwindCss />
<main>
  <div class="h-screen flex items-center bg-gray-100 justify-center">
    <div class="drop-shadow-lg p-4 bg-white rounded">
      <div
        id="connection"
        class="bg-green-500 p-1 text-white font-bold border-4 border-black text-center text-lg"
      >
        Thanks for using Listen.moe!
      </div>

      <div class="font-bold text-lg">
        Listen.moe - It's time to ditch other radios.
      </div>
      <div class="bg-zinc-800 text-white p-1 text-center">
        Now playing:
        <div class="italic font-bold text-center">
          <!-- svelte-ignore a11y-distracting-elements -->
          <marquee id="currentSong">{currentSong}</marquee>
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
      <div class="flex text-center">
        <div hidden class="" id="audio">
          Looks like you are using opera! For some reason the player breaks in
          Opera, please use this:
          <div>
            <audio id="audioPlayer">
              <source src="https://listen.moe/fallback" type="audio/mpeg" />
            </audio>
          </div>
        </div>
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
    </div>
  </div>
</main>

<style>
  :root {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
  }
</style>
