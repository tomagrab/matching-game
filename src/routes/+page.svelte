<script lang="ts">
  import { emoji } from "$lib/Emoji";

  type State = "start" | "playing" | "paused" | "won" | "lost";

  let state: State = "start";

  let size = 20;
  let grid = createGrid();
  let maxMatches = grid.length / 2;
  let selected: number[] = [];
  let matches: string[] = [];
  let timerID: number | null = null;
  let time = 60;

  function startGameTimer() {
    function countdown() {
      state !== "paused" && (time -= 1);
    }

    timerID = setInterval(countdown, 1000);
  }

  function createGrid() {
    let cards = new Set<string>();
    let maxSize = size / 2;

    while (cards.size < maxSize) {
      const randonIndex = Math.floor(Math.random() * emoji.length);
      cards.add(emoji[randonIndex]);
    }

    return shuffle([...cards, ...cards]);
  }

  function shuffle<Items>(array: Items[]) {
    return array.sort(() => Math.random() - 0.5);
  }

  function selectCard(cardIndex: number) {
    selected = selected.concat(cardIndex);
  }

  function matchCards() {
    const [first, second] = selected;

    if (grid[first] === grid[second]) {
      matches = matches.concat(grid[first]);
    }

    setTimeout(() => {
      selected = [];
    }, 300);
  }

  function resetGame() {
    timerID && clearInterval(timerID);
    grid = createGrid();
    maxMatches = grid.length / 2;
    selected = [];
    matches = [];
    timerID = null;
    time = 60;
  }

  function gameWon() {
    state = "won";
    resetGame();
  }

  function gameLost() {
    state = "lost";
    resetGame();
  }

  function pauseGame(e: KeyboardEvent) {
    if (e.key === "Escape") {
      switch (state) {
        case "playing":
          state = "paused";
          break;
        case "paused":
          state = "playing";
          break;
      }
    }
  }

  $: if (state === "playing") {
    !timerID && startGameTimer();
  }

  $: selected.length === 2 && matchCards();
  $: maxMatches === matches.length && gameWon();
  $: time === 0 && gameLost();
  $: console.log({ state, selected, matches });
</script>

<svelte:window on:keydown={pauseGame} />

{#if state === "paused"}
  <h1 class="mb-4">Game paused</h1>
  <button on:click={() => (state = "playing")} class="btn btn-primary"
    >Resume</button
  >
{/if}

{#if state === "start"}
  <h1 class="text-4xl">Matching game</h1>
  <p class="my-4">Click on the cards to find the matching pairs</p>
  <button on:click={() => (state = "playing")} class="btn btn-primary"
    >Start</button
  >
{/if}

{#if state === "playing"}
  <h1 class="mb-4 text-4xl animate-pulse">{time}</h1>
  <div class="matches">
    {#each matches as card}
      <div class="match">{card}</div>
    {/each}
  </div>
  <div class="grid grid-cols-5 gap-4">
    {#each grid as card, index}
      {@const isSelected = selected.includes(index)}
      {@const isSelectedOrMatched =
        selected.includes(index) || matches.includes(card)}
      {@const isMatched = matches.includes(card)}

      <button
        class="bg-slate-700 border-4 rounded transition-opacity ease-out duration-200 card"
        class:border-slate-700={!isSelectedOrMatched}
        class:border-pink-400={isSelected}
        class:border-blue-400={isMatched}
        class:opacity-40={isMatched}
        class:flip={isSelectedOrMatched}
        on:click={() => selectCard(index)}
      >
        <div
          class="flex flex-col items-center justify-center p-4 rounded"
          class:back={!isSelectedOrMatched}
        >
          {card}
        </div>
      </button>
    {/each}
  </div>
{/if}

{#if state === "won"}
  <h1>You won!</h1>
  <button on:click={() => (state = "start")} class="btn btn-primary"
    >Play again</button
  >
{/if}

{#if state === "lost"}
  <h1>You lost!</h1>
  <button on:click={() => (state = "start")} class="btn btn-secondary"
    >Play again</button
  >
{/if}

<style lang="scss">
  .card {
    height: 140px;
    width: 140px;
    font-size: 4rem;
    transform-style: preserve-3d;
    transition: rotate 0.3s ease-out;

    &.flip {
      rotate: y 180deg;
      pointer-events: none;
    }

    & .back {
      position: absolute;
      inset: 0;
      display: grid;
      place-content: center;
      backface-visibility: hidden;
      rotate: y 180deg;
    }
  }

  .matches {
    font-size: 4rem;
    display: flex;
  }
</style>
