<template>
  <div>
    <div class="title" :style="{left: `${state.left}%`, top: `${state.top}%`}">
      <div>
        fuck®2fa™
      </div>
    </div>
    <div v-if="state.prompt">{{ state.prompt }}</div>
    <div v-else>
      <div class="code">{{ state.code }}</div>
      <div class="progress-bar">
        <div class="bar" :style="{width: `${state.progress}%`}"></div>
      </div>
      <div class="next-code">
        <span class="next-code-label">next</span>
        {{ state.nextCode }}
      </div>
    </div>
    <div class="copyright">©2022 <a href="https://447f.misaka.org">447f.misaka.org</a></div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, reactive } from 'vue'

const state = reactive({
  prompt: '',
  code: '',
  nextCode: '',
  progress: 0,
  left: 0,
  top: 0
})
state.prompt = ''
const searchValue = new URLSearchParams(location.search)
const secretKey = searchValue.get('key')
if (!secretKey) {
  state.prompt = 'You have no secret key specified. Specify using query parameter "key".'
}
const generate = () => {
  // @ts-ignore optlib does not support typescript
  const mfa = window?.otplib?.authenticator
  if (!mfa) state.prompt = 'otplib Failure. Consider refresh.'
  if (!secretKey) return
  state.progress = (mfa.timeRemaining() - 1) / 29 * 100
  if (mfa.timeRemaining() % 3 === 0) {
    state.left = Math.random() * 100
    state.top = Math.random() * 100
  }
  if (!state.code || mfa.timeUsed() === 0) {
    state.code = mfa.generate(secretKey)
    mfa.options = { epoch: new Date().getTime() + 30000 }
    state.nextCode = mfa.generate(secretKey)
    mfa.resetOptions()
  }
}
onMounted(() => {
  setInterval(generate, 1000)
  generate()
})
</script>

<style lang="scss">
body {
  margin: 0;
}

#app {
  font-family: Consolas, monospace, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #6cf;
  background-color: #012;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

:link, :visited {
  color: #9cf;
  &:hover {
    color: #6cf;
  }
}
</style>
<style scoped lang="scss">
.title {
  position: fixed;
  opacity: 0.3;
  font-size: 120px;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 0;
  height: 0;
}

.code {
  font-size: 72px;
  line-height: 80px;
}

.next-code {
  font-size: 36px;
  line-height: 72px;

  .next-code-label {
    font-size: 24px;
  }
}

.progress-bar {
  height: 4px;
  background-color: #234;

  .bar {
    height: 100%;
    background-color: #6cf;
  }
}
</style>
