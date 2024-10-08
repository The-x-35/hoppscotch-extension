<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { DEFAULT_ORIGIN_LIST } from "./defaultOrigins"


const origins = ref<string[]>([])
const inputText = ref('')
const placeholderURL = "https://hoppscotch.io"
const errorMessage = ref('')

const getOriginList = () =>
  new Promise((resolve) => {
    chrome.storage.sync.get(["originList"], async (items) => {
      if (!items || !items.originList) {
        await storeOriginList(DEFAULT_ORIGIN_LIST)
        resolve(DEFAULT_ORIGIN_LIST)
      } else {
        resolve(JSON.parse(items.originList))
      }
    })
  })

const storeOriginList = (originList: string[]) =>
  new Promise<void>((resolve) => {
    chrome.storage.sync.set(
      {
        originList: JSON.stringify(originList),
      },
      () => {
        resolve()
      }
    )
  })

const onAddClick = async (event: { preventDefault: () => void; }) => {
  event.preventDefault()
  try {
    const parsedURL = new URL(inputText.value)
    if (origins.value.includes(parsedURL.origin)) {
      errorMessage.value = "Origin is already on the list"
    } else {
      origins.value.push(parsedURL.origin)
      inputText.value = ""
      await storeOriginList(origins.value)
      errorMessage.value = ""
    }
  } catch (e) {
    errorMessage.value = "Improper URL"
  }
}

const onInputTextChange = (ev:any) => {
  inputText.value = ev.target.value
  errorMessage.value = ""
}

const onDeleteOriginClicked = async (index: number) => {
  origins.value.splice(index, 1)
  await storeOriginList(origins.value)
}

onMounted(() => {
  getOriginList()
    .then((list:any) => {
      origins.value = list
    })
    .catch(() => {
      // Just fail silently :P
    })

  chrome.tabs.query({ active: true }, (result) => {
    if (result.length > 0) {
      try {
        if (result[0].url && result[0].url.startsWith("http")) {
          const url = new URL(result[0].url)
          if (url && url.origin) {
            inputText.value = url.origin
          }
        }
      } catch (_e) {}
    }
  })
})
</script>

<template>
  <div>
    <form novalidate class="origin-input-box">
      <label class="origin-input-label" for="origin-input">Enter new origin</label>
      <div class="origin-input-wrapper">
        <input id="origin-input" required :placeholder="placeholderURL" class="origin-input" v-model="inputText" @input="(ev: Event) => onInputTextChange(ev)">
        <button class="origin-add" type="submit" @click="onAddClick">
          <img src="./add-icon.svg" alt="Add Icon">
          <span class="button-text">Add</span>
        </button>
      </div>
    </form>
    <div v-if="errorMessage" class="err">
      <img src="./error-icon.svg" alt="Error Icon">
      <span class="err-text">{{ errorMessage }}</span>
    </div>
    <label class="origin-input-label">Active origins</label>
    <ul class="origin-list">
      <li v-for="(origin, index) in origins" :key="index" class="origin-list-entry">
        <span class="origin-list-entry-origin">{{ origin }}</span>
        <button
          class="origin-delete"
          :disabled="origin === 'https://hoppscotch.io'"
          @click="onDeleteOriginClicked(index)"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            height="18"
            viewBox="0 0 24 24"
            fill="currentColor"
            width="18"
            :style="{ color: origin === 'https://hoppscotch.io' ? '#ffffff55' : '#fe4645' }"
          >
            <path d="M0 0h24v24H0z" fill="none"/>
            <path d="M6 19c0 1.1.9 2 2 2h8c1.1 0 2-.9 2-2V7H6v12zM19 4h-3.5l-1-1h-5l-1 1H5v2h14V4z"/>
          </svg>
        </button>
      </li>
    </ul>
  </div>
</template>

<style scoped>
@font-face {
  font-family: "Inter", sans-serif;
  font-style: normal;
  font-weight: 400;
  font-display: swap;
  src: url("./Inter-Regular.ttf") format("truetype");
}

@font-face {
  font-family: "Inter", sans-serif;
  font-style: normal;
  font-weight: 500;
  font-display: swap;
  src: url("./Inter-Bold.ttf") format("truetype");
}

body {
  background-color: #202124;
  width: 30rem;
  height: 35rem;
  color: white;
  font-family: "Inter", sans-serif;
}

.err {
  background-color: #fe4645bb;
  color: white;
  display: flex;
  padding: 0.4rem;
  border-radius: 3px;
}

.err-text {
  padding: 0 0.4rem;
}

.origin-input:focus,
.origin-input:hover {
  box-shadow: inset 0 0 0 2px rgba(255, 255, 255, 0.5);
  outline: none;
}

.origin-input {
  padding: 4px 8px;
  background-color: rgba(0, 0, 0, 0.2);
  color: white;
  border: 0;
  border-radius: 2px;
  font-family: "Inter", sans-serif;
  display: flex;
  flex: 1;
}

.origin-input-wrapper {
  display: flex;
}

.origin-input-label {
  color: rgba(255, 255, 255, 0.5);
  margin: 8px;
  display: flex;
  font-size: 0.7rem;
}

.origin-list {
  display: flex;
  flex-direction: column;
  list-style-type: none;
  padding: 0;
  margin: 0;
  overflow: auto;
  max-height: 500px;
}

.origin-add {
  background-color: #31c48d;
  color: rgba(0, 0, 0, 0.82);
  border-radius: 2px;
  padding: 4px 12px;
  outline: none;
  font-weight: bold;
  border: 0;
  margin-left: 4px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
}

.origin-delete {
  cursor: pointer;
  background-color: transparent;
  color: #fe4645;
  border-radius: 2px;
  padding: 4px 12px;
  outline: none;
  border: 0;
  margin-left: 4px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

.origin-delete:disabled {
  color: #ffffff55;
  cursor: auto;
}

.button-text {
  margin-left: 2px;
}

.origin-list-entry {
  display: flex;
  padding: 4px 0;
  align-items: center;
  margin-left: 8px;
}

.origin-list-entry-origin {
  flex: 1;
  display: flex;
}
</style>
