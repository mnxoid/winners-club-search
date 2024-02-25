<script setup lang="ts">
import { computed, reactive } from 'vue'


interface Item {
  category: string
  title: string
  address: string
}

var data: Item[] = []

const is_rus = document.location.href.includes('russian')

const url = is_rus ? 'https://cdn.sttwins.com/static/book/data_ru.json' : 'https://cdn.sttwins.com/static/book/data_ua_new.json'

fetch(url)
  .then(res => res.json())
  .then(out => {
    data = Object.keys(out)
      .flatMap(category => 
        Object.keys(out[category])
          .flatMap(subset => 
            out[category][subset]
              .map((item: any) => ({category, ...item}))
          )
      )
    filterItems()
  })
  .catch(err => { throw err });

const directions: string[] = is_rus ? ['СЗ', 'С', 'СВ', 'З', 'Ц', 'В', 'ЮЗ', 'Ю', 'ЮВ', 'П'] : ['РІ', 'ДХ', 'ПЛ', 'МІ', 'КО', 'ДА', 'СГ', 'ПФ', 'АП', 'ОМ', 'ПР']
const active_directions: string[] = reactive([])

function toggleDirection(direction: string) {
  const index = active_directions.indexOf(direction)
  if (index === -1) {
    active_directions.push(direction)
  } else {
    active_directions.splice(index, 1)
  }
  filterItems()
}

function setOnlyDirection(direction: string) {
  active_directions.splice(0, active_directions.length)
  active_directions.push(direction)
  filterItems()
}

var filtered_items: Item[] = reactive([])

var search_text: string = ""

function filterItems() {
  const txt = search_text.toLowerCase();
  const selected_directions = active_directions;

  let test = (item: Item, srctxt: string) => (
      ((srctxt.length === 0) || item.title.toLowerCase().includes(srctxt) || item.address.toLowerCase().includes(srctxt)) &&
      (selected_directions.length === 0 || selected_directions.includes(item.address.split('-')[0]))
    )

  let filter_results = data.filter(item => {
    return test(item, txt) || test(item, transliterate(txt))
  })

  filtered_items.splice(0, filtered_items.length)
  filter_results.forEach(item => filtered_items.push(item))
}

function transliterate(text: string) {
    const transliterationMap: Record<string, string> = {
        'a': 'а', 'b': 'б', 'c': 'ц', 'd': 'д', 'e': 'е', 'f': 'ф', 'g': 'ґ', 'h': 'г', 'i': 'і', 'j': 'й', 'k': 'к', 'l': 'л',
        'm': 'м', 'n': 'н', 'o': 'о', 'p': 'п', 'q': 'к', 'r': 'р', 's': 'с', 't': 'т', 'u': 'у', 'v': 'в', 'w': 'в', 'x': 'кс',
        'y': 'и', 'z': 'з',

        'A': 'А', 'B': 'Б', 'C': 'Ц', 'D': 'Д', 'E': 'Е', 'F': 'Ф', 'G': 'Ґ', 'H': 'Г', 'I': 'І', 'J': 'Й', 'K': 'К', 'L': 'Л',
        'M': 'М', 'N': 'Н', 'O': 'О', 'P': 'П', 'Q': 'К', 'R': 'Р', 'S': 'С', 'T': 'Т', 'U': 'У', 'V': 'В', 'W': 'В', 'X': 'КС',
        'Y': 'И', 'Z': 'З',

        ' ': ' ', // space
        '.': '.', ',': ',', '!': '!', '?': '?', "'": "'", '"': '"', ';': ';', ':': ':', '(': '(', ')': ')', '[': '[', ']': ']',
        '{': '{', '}': '}', '-': '-', '_': '_', '+': '+', '=': '=', '*': '*', '/': '/', '\\': '\\', '|': '|', '~': '~', '`': '`',
        '<': '<', '>': '>', '@': '@', '#': '#', '$': '$', '%': '%', '^': '^', '&': '&'
    };

    return text.split('').map(char => transliterationMap[char] || char).join('');
}
</script>

<template>
  <main>
    <div id="search">
      <input 
        type="text" 
        placeholder="Search..." 
        name="search_address"
        v-model="search_text"
        @input="filterItems"
      >
      <div id="filters">
        <div 
          v-for="direction in directions"
          :key="direction"
          class="square" 
          :class="{ 'active-square': active_directions.includes(direction) }"
          @click.exact="toggleDirection(direction)"
          @click.shift="setOnlyDirection(direction)"
        >{{ direction }}</div>
      </div>
    </div>
    <div id="results">
      <div
        class="card"
        v-for="item in filtered_items"
        :key="item.toString()"
      >
        <h2>{{ item.title }}</h2>
        <p>{{ item.address }}</p>
        <p>({{ item.category }})</p>
      </div>
    </div>
  </main>
</template>

<style scoped>
  main {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 100%;
  }

  #search {
    margin-bottom: 2rem;
    width: 100%;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
  }

  #search > input {
    flex-grow: 1;
    font-size: 30px;
    margin-right: 50px;
  }

  #filters {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
  }

  #results {
    width: 90%;
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
  }

  .card {
    font-size: 2em;
    width: 300px;
    margin: 5px;
  }

  .square {
    font-size: 30px;
    display: flex;
    width: 100px;
    height: 100px;
    align-items: center;
    justify-content: center;
    border: 1px solid #2929295e;
    margin: 5px;
  }

  .active-square {
    border-color: #007bff;
  }

  @media (max-width: 1250px) {
    #search {
      display: initial;
    }

    #search > input {
      width: 100%;
      font-size: 12px;
      margin: 0;
    }

    #filters {
      display: flex;
      flex-direction: row;
      flex-wrap: wrap;
    }

    .square {
      font-size: 10px;
      display: flex;
      width: 20px;
      height: 20px;
      margin: 5px;
    }

    .card {
      font-size: 12px;
      width: 100%;
      margin: 5px;
      display: flex;
      flex-direction: row;
      align-items: center;
    }

    .card > h2 {
      font-size: 24px;
      flex-grow: 1;
    }

    .card > p {
      flex-grow: 1;
    }
  }
</style>