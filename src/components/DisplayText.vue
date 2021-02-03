<template>
  <div v-show="displayTextVisible" class="flex h-screen">
    <div class="m-auto bg-red-500">
      <div class="p-20 text-white text-8xl">{{ textToDisplay }}</div>
    </div>
  </div>
</template>
<script lang="ts">
import { defineComponent, ref, onMounted, onBeforeUnmount } from "vue";
import axios from "axios";

export default defineComponent({
  name: "DisplayText",
  setup() {
    const textToDisplay = ref("");
    let pollTimer: number;
    let lastCommandGiven = "";
    const intervalTime = 5000; // in miliseconds
    const tickToDisplay = 2; // on tick is one interval time length
    let currentTickDisplayed = 0;
    const displayTextVisible = ref(false);

    const whatText = (command: string) => {
      switch (command) {
        case "sayhi":
          return "hi";
        case "sayboo":
          return "boo";
        default:
          return "";
      }
    };

    const getLatestCommand = () => {
      console.log("1 tick");
      axios
        .get("https://cutie-backend.glitch.me/lastCommand")
        .then(response => {
          console.log(JSON.stringify(response.data));
          if (lastCommandGiven === response.data.lastCommand) {
            currentTickDisplayed++;
            if (currentTickDisplayed > tickToDisplay) {
              displayTextVisible.value = false;
            }
            console.log("no change " + lastCommandGiven);
          } else {
            console.log("change " + lastCommandGiven);
            lastCommandGiven = response.data.lastCommand;
            currentTickDisplayed = 0;
            textToDisplay.value = whatText(lastCommandGiven);
            if (textToDisplay.value === "") {
              displayTextVisible.value = false;
            } else {
              displayTextVisible.value = true;
            }
          }
        });
    };

    onMounted(() => {
      pollTimer = setInterval(() => {
        getLatestCommand();
      }, intervalTime);
    });

    onBeforeUnmount(() => {
      clearInterval(pollTimer);
      displayTextVisible.value = false;
    });

    // expose to template
    return {
      textToDisplay,
      displayTextVisible
    };
  }
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
