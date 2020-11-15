<template>
  <div id="app">
    <h1>Simon Says</h1>
    <div class="container">
      <div class="panel">
        <div
          class="panel-item left-top"
          @click.prevent="panelClicked($event.currentTarget)"
          data-audio="1"
        ></div>
        <div
          class="panel-item right-top"
          @click.prevent="panelClicked($event.currentTarget)"
          data-audio="2"
        ></div>
        <div
          class="panel-item left-bottom"
          @click.prevent="panelClicked($event.currentTarget)"
          data-audio="3"
        ></div>
        <div
          class="panel-item right-bottom"
          @click.prevent="panelClicked($event.currentTarget)"
          data-audio="4"
        ></div>
      </div>
      <div class="buttons">
        <h2 class="heading" v-if="say !== ''">
          {{ say === "simon" ? "Саймон говорит" : "Повторите за Саймоном" }}
        </h2>
        <button class="btn" type="button" @click="toggleModal">Правила</button>
        <button class="btn" type="button" @click="startGame" v-if="!counter">
          Начать
        </button>
        <h2 class="heading" v-if="counter">Раунд {{ counter }}</h2>
        <div>
          <h2 class="heading">Сложность:</h2>
          <div>
            <input
              type="radio"
              name="complexity"
              id="easy"
              v-model="complexity"
              value="1500"
            />
            <label for="easy">Легко</label>
          </div>
          <div>
            <input
              type="radio"
              name="complexity"
              id="hard"
              v-model="complexity"
              value="1000"
            />
            <label for="hard">Сложно</label>
          </div>
          <div>
            <input
              type="radio"
              name="complexity"
              id="very-hard"
              v-model="complexity"
              value="400"
            />
            <label for="very-hard">Очень сложно</label>
          </div>
        </div>
        <div class="modal" @click="toggleModal">
          <div class="modal-body">
            <h2>Правила игры</h2>
            <hr />
            <p>
              Раунд в игре состоит в том, что Саймон говорит (освещает кнопки в
              случайном порядке), после чего Вам нужно повторить
              последовательность кнопок. С каждым новым раундом Саймон говорит
              больше. В случае не совпадения игра заканчивается
            </p>
          </div>
        </div>
      </div>
      <div class="audio"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data: () => ({
    canClick: false,
    panelItems: [],
    sequenseToGuess: [],
    randomPanels: [],
    counter: 0,
    complexity: 1500,
    modal: false,
    audio: 1,
    say: "",
  }),
  methods: {
    toggleModal() {
      document.querySelector(".modal").classList.toggle("active");
    },

    getAudio(index) {
      let audio = document.querySelector(".audio");

      audio.innerHTML = "";

      audio.insertAdjacentHTML(
        "afterbegin",
        `<audio autoplay>
            <source src="audio/${index}.ogg" type="audio/ogg" />
            <source src="audio/${index}.mp3" type="audio/mp3" />
          </audio>`
      );
    },

    getPanelItems() {
      const leftTop = document.querySelector(".left-top");
      const rightTop = document.querySelector(".right-top");
      const leftBottom = document.querySelector(".left-bottom");
      const rightBottom = document.querySelector(".right-bottom");

      this.panelItems.push(leftTop, rightTop, leftBottom, rightBottom);
    },

    flash(panel) {
      return new Promise((resolve) => {
        panel.className += " active";

        this.getAudio(panel.dataset.audio);

        setTimeout(() => {
          panel.className = panel.className.replace("active", "");

          setTimeout(() => {
            resolve();
          }, 500);
        }, +this.complexity);
      });
    },

    panelClicked(panel) {
      if (!this.canClick) return;

      this.getAudio(panel.dataset.audio);

      let tmp = this.sequenseToGuess.shift();

      if (tmp === panel) {
        if (this.sequenseToGuess.length === 0) {
          this.randomPanels.push(this.getRandomPanels(this.panelItems));

          setTimeout(() => {
            this.startGame();
          }, 500);
        }
      } else {
        alert(`Вы проиграли на уровне ${this.counter}`);
        this.randomPanels = [];
        this.randomPanels.push(this.getRandomPanels(this.panelItems));
        this.counter = 0;
        this.canClick = false;
        this.say = "";
      }
    },

    getRandomPanels(panels) {
      return panels[parseInt(Math.random() * panels.length)];
    },

    async startGame() {
      this.canClick = false;
      this.sequenseToGuess = [...this.randomPanels];
      this.counter += 1;
      this.say = "simon";

      for (const panel of this.randomPanels) {
        await this.flash(panel);
      }

      this.canClick = true;
      this.say = "player";
    },
  },
  mounted() {
    this.getPanelItems();
    this.randomPanels.push(this.getRandomPanels(this.panelItems));
  },
};
</script>

<style>
body {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  margin-top: 60px;
  background-color: #fafafa;
}

h1 {
  text-align: center;
}

.container {
  max-width: 80%;
  margin: 0 auto;
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
  align-items: center;
}

.modal {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(5px);
  display: flex;
  align-items: center;
  justify-content: center;
  transform: scale(0);
  transition: 0.2s;
}

.modal.active {
  transform: scale(1);
}

.modal-body {
  background-color: #fff;
  border: 1px solid #ddd;
  border-radius: 5px;
  padding: 1rem;
  max-width: 400px;
  box-shadow: 0 5px 25px rgba(54, 54, 54, 0.7);
  margin: 0 1rem;
}

.heading,
.buttons {
  margin-top: 1rem;
  margin-bottom: 0.5rem;
}

label {
  font-size: 1.1rem;
  margin-left: 0.3rem;
}

.btn {
  color: #fff;
  background-color: rgb(69, 69, 228);
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  font-size: 1.1rem;
  margin-right: 1rem;
}

.panel {
  width: 30vw;
  height: 30vw;
  display: flex;
  flex-wrap: wrap;
  border-radius: 100%;
  box-shadow: 5px 5px 25px rgba(54, 54, 54, 0.6);
  background-color: rgb(136, 136, 136);
}

@media (max-width: 600px) {
  .panel {
    width: 50vw;
    height: 50vw;
  }
}

.panel-item {
  width: 50%;
  height: 50%;
  opacity: 0.8;
}

.panel-item.active {
  background-color: #000;
  opacity: 1;
}

.panel-item:active {
  transform: scale(0.97);
  cursor: pointer;
}

.panel-item:hover {
  opacity: 1;
  cursor: pointer;
}

.left-top {
  border-top-left-radius: 100%;
  background-color: rgb(255, 50, 50);
}

.right-top {
  border-top-right-radius: 100%;
  background-color: rgb(50, 50, 255);
}

.left-bottom {
  border-bottom-left-radius: 100%;
  background-color: rgb(50, 255, 50);
}

.right-bottom {
  border-bottom-right-radius: 100%;
  background-color: rgb(255, 255, 50);
}
</style>
