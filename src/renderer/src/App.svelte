<script lang="ts">
  import { MonitorX, EyeOff } from 'lucide-svelte'
  import icon from '../../../resources/icon.png'

  let ipc = window.electron.ipcRenderer
  let window_name = document.title

  let isYesHover, realQuestion, clickedYes, maxClickCount, currentClickCount, byeMsg
  realQuestion = 'Есть ли зверь сильнее мухи?'
  byeMsg = 'Так я и знал'

  currentClickCount = 0
  maxClickCount = 0
  while (maxClickCount < 20) {
    maxClickCount = Math.floor(Math.random() * 60)
  }
  console.log(maxClickCount)

  function quit(): void {
    ipc.send('quit')
  }
  function minimize(): void {
    ipc.send('minimize')
  }

  function yesHover(): void {
    isYesHover = true
  }
  function yesUnHover(): void {
    if (!clickedYes) {
      isYesHover = false
    }
  }
  function yesClick(): void {
    clickedYes = true
    setTimeout(() => {
      quit()
    }, 3000)
  }

  function noClick(event): void {
    if (currentClickCount >= maxClickCount) {
      byeMsg = 'Ладно, ты победил'
      yesClick()
      return
    }
    currentClickCount++
    let maxX =
      document.querySelector('.mainwindow').getBoundingClientRect().width -
      event.target.getBoundingClientRect().width
    let maxY =
      document.querySelector('.mainwindow').getBoundingClientRect().height -
      event.target.getBoundingClientRect().height * 1.5

    let randomX = Math.floor(Math.random() * maxX)
    let randomY =
      Math.floor(Math.random() * maxY) -
      document.querySelector('.mainwindow>.text').getBoundingClientRect().height

    console.log(document.querySelector('.mainwindow>.text').getBoundingClientRect().height)

    event.target.style.left = randomX + 'px'
    event.target.style.top = randomY + 'px'
  }
</script>

<div class="dragbar">
  <div class="text">
    <img src={icon} alt="app icon" class="icon" />
    <span class="divider"></span>
    <p>{window_name}</p>
  </div>
  <div class="buttons">
    <span class="divider"></span>
    <button class="minimize" on:click|stopPropagation={minimize}><EyeOff /></button>
    <span class="divider"></span>
    <button class="close" on:click|stopPropagation={quit}><MonitorX /></button>
  </div>
</div>

<div class="mainwindow">
  <div class="text">
    <h1 class="title">Вопрос:</h1>
    <span class="question">{isYesHover ? 'Сосал?' : realQuestion}</span>
  </div>
  <div class="buttons">
    <button on:mouseenter={yesHover} on:mouseleave={yesUnHover} on:click|capture={yesClick}>
      Да
    </button>
    <button class="filler"></button>
    <!-- eslint-disable-next-line prettier/prettier -->
    <button class="no" on:click={noClick} style="left: calc(50% + 1rem);"> Нет </button>
  </div>
</div>

<div class="bye" class:visible={clickedYes}>
  <h1>{byeMsg}</h1>
</div>

<style lang="scss">
  .mainwindow {
    height: calc(100dvh - 2rem - 1px);
    width: 100dvw;

    .text {
      display: flex;
      flex-direction: column;
      align-items: center;

      padding: 0 0.8rem;

      .title {
        position: relative;
        margin-bottom: -1.2rem;

        font-weight: 700;
        font-size: 3rem;

        color: transparent;
        background: linear-gradient(to bottom, #eee 40%, #999 70%);
        background-clip: text;

        text-wrap: nowrap;
      }
      .question {
        position: relative;
        font-size: 2.8rem;

        color: transparent;
        background: linear-gradient(to bottom, #eee 60%, #aaa 80%);
        background-clip: text;

        &::after {
          content: '';
          position: absolute;
          bottom: -0.8rem;
          left: 2.5%;
          background: url('./assets/highlight.svg');
          width: 95%;
          height: 1.5rem;
          z-index: -1;
        }
      }
      @media (max-width: 720px) {
        .title {
          font-size: 2.8rem;
        }

        .question {
          font-size: 2rem;
        }
      }
    }
    .buttons {
      display: flex;
      justify-content: center;
      gap: 1rem;
      position: relative;
      margin-top: 1rem;

      button {
        border: none;
        padding: 0.5rem;
        width: 5rem;
        border-radius: 100rem;

        background: var(--color-background-lightest);
        color: var(--color-text-primary);
        outline: #fff6 0 solid;

        transition: outline 100ms ease-in;

        &:hover {
          outline: #fff6 0.4rem solid;
        }

        &.filler {
          background-color: transparent;
          &:hover {
            outline: none;
          }
        }

        &.no {
          position: absolute;
        }
      }
    }
  }
  .bye {
    -webkit-app-region: drag;
    position: absolute;
    height: calc(100dvh - 2rem - 1px);
    width: 100dvw;
    left: 0;
    top: 0;
    margin-top: calc(2rem + 1px);
    background-color: var(--color-background);
    display: flex;
    justify-content: center;
    align-items: center;

    visibility: hidden;
    opacity: 0;

    transition: opacity 1s;

    &.visible {
      visibility: visible;
      opacity: 1;
    }

    h1 {
      font-weight: 600;
      color: transparent;
      background: linear-gradient(to bottom, #eee 60%, #bbb 80%);
      background-clip: text;
      text-rendering: optimizeSpeed;
      transform: translateY(-1rem);
    }
  }
</style>
