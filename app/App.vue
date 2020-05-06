<template>
  <div class="absolute-fill">
    <div
      v-for="word in wordsList"
      :key="word.value"
      :style="{
        color: 'yellow',
        position: 'absolute',
        right: word.x,
        top: word.y,
      }"
    >
      {{ word.value }}
    </div>
  </div>
</template>

<script lang="ts">
import { interval, fromEvent } from "rxjs";
import { throttle, map, flatMap, distinctUntilChanged } from "rxjs/operators";

async function* nextWord() {
  while (true) {
    const response = await fetch(
      "https://random-word-api.herokuapp.com/word?number=100"
    );
    const words = await response.json();
    yield* words;
  }
}

export default {
  data: () => ({
    iter: nextWord(),
    wordsList: [],
    timer$: interval(1000),
    keyDown$: fromEvent(document, "keydown").pipe(
      (distinctUntilChanged(
        (x: KeyboardEvent, y: KeyboardEvent) => x.type === y.type
      ),
      map((x: KeyboardEvent) => x.key))
    ),
  }),

  async mounted() {
    const word = await this.iter.next();
    this.wordsList = [
      ...this.wordsList,
      {
        value: word.value,
        x: 0,
        y: 200,
      },
    ];

    this.timer$.subscribe(() => {
      for (const word of this.wordsList) {
        word.x += 20;
      }
    });

    this.timer$
      .pipe(
        throttle(() => interval(4000)),
        flatMap(async () => {
          const word = await this.iter.next();
          return word.value;
        })
      )
      .subscribe((word) => {
        this.wordsList = [
          ...this.wordsList,
          {
            value: word,
            x: 0,
            y: Math.floor(Math.random() * window.innerHeight),
          },
        ];
      });

    this.keyDown$.subscribe((x) => {
      console.log(x);
    });
  },
};
</script>

<style scoped>
.absolute-fill {
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  background: blue;
  position: absolute;
}
</style>
