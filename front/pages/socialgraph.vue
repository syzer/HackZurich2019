<template>
  <v-layout>
    <v-flex class="text-center">
      <v-row align="center" justify="center">
        <v-img
          src="https://github.com/syzer/HackZurich2019/raw/master/data/wireframes/ComuteHeatmap.jpg"
          lazy-src="https://github.com/syzer/HackZurich2019/raw/master/data/wireframes/ComuteHeatmap.jpg"
          aspect-ratio="0.3"
          class="grey lighten-2"
          max-width="500"
          max-height="620"
        ></v-img>
      </v-row>
    </v-flex>
  </v-layout>
</template>
<style>
  :root {
    --active-color: #c2751a;

    --status-green: #60AD08;
    --status-green-hover: #315903;
    --status-red: #EE0000;
    --status-red-hover: #BA0000;
    --status-yellow: #F6AD2B;
    --status-yellow-hover: #D98D06;
    --status-orange: #FF6600;
    --status-gray: #CFCAC7;

    --migros-orange: #ff6600;
    --migros-orange-hover: #DE450A;
    --text-black: #333333;
    --border-gray: #CFCAC7;
    --label-small-gray: #BBBBBB;
    --label-big-gray: #676767;
    --background-darker: #E7E5E3;
    --background-lighter: #F1EFEE;
  }

  body {
    /*padding: 50px;*/
    font: 14px "Lucida Grande", Helvetica, Arial, sans-serif;
    background-color: var(--background-lighter);
  }

  a {
    color: #00B7FF;
  }

  .link {
    stroke: var(--label-small-gray);
    stroke-width: 2;
  }

  .node {
    cursor: move;
  }

  .node text {
    font-family: HelveticaNeue, sans-serif;
    font-size: 22px;
    font-weight: bold;
  }

  .node > rect {
    fill: var(--label-small-gray);
  }

  .node > .active {
    fill: var(--active-color);
  }

  .node .name {
    pointer-events: none;
  }

  .description:hover {
    fill: var(--active-color);
  }

  .description {
    fill: white;
    background-color: var(--migros-orange);
  }

  .Rectangle {
    width: 327px;
    height: 70px;
    border-radius: 5px;
    background-color: var(--background-lighter);
    border: solid 1px var(--border-gray);
  }

</style>
