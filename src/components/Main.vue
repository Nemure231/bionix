<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';

const trixText = ref('')
const fixation = ref(5)
const menu = ref(false)
const result = ref('')

let splitMulti = (str, tokens) => {
  var tempChar = tokens[0];
  for (var i = 1; i < tokens.length; i++) {
    str = str.split(tokens[i]).join(tempChar);
  }
  str = str.split(tempChar);
  return str;
}

let setTextToTrix = () => {
  trixText.value = document.getElementById("editor").value;

}

let convert = () => {
  setTimeout(() => {
    const splitMultiResult = splitMulti(`${trixText.value}`, ['<p>', '</p>'])
    const odd = ref([])
    const even = ref([])
    splitMultiResult.forEach((x, index) => {
      if ((x) % 2 == 0) {
        odd.value.push(index)
      } else {
        even.value.push(index)
      }
    });

    odd.value.forEach(e => {
      splitMultiResult.splice(e, 1, '</p><p>')
    });

    const newArray = ref([])
    even.value.forEach(e => {
      const spi = splitMultiResult[e].split(' ')

      newArray.value.push(spi)

    });

    var newArray2 = ref([])

    const fixationPerc = fixation.value / parseInt(10)


    for (let ex = 0; ex < newArray.value.length; ex++) {
      var newArray3 = ref([])
      newArray.value[ex].forEach((e, inde) => {

        const fixPoint = Math.ceil(fixationPerc * e.length)
        const ee = e.replace(/&nbsp;/g, ' ').replace('<br>', ' ').trim()
        const splitMultiResult = `<strong class="font-bold">${ee.slice(0, fixPoint)}</strong>${ee.slice(fixPoint)}`

        newArray3.value[inde] = splitMultiResult
      })

      const initReduce = '<p>'
      const sumReduce = newArray3.value.reduce(
        (prev, curr) => prev + curr + ' ',
        initReduce
      )
      const aum = `${sumReduce}</p>`

      newArray2.value[ex] = aum
    }

    const join = newArray2.value.join(' ')

    result.value = join

  }, 50)
}


let trixStyle = () => {
  Trix.config.blockAttributes.default.tagName = "p"
  Trix.config.blockAttributes.default.breakOnReturn = true;

  Trix.Block.prototype.breaksOnReturn = function () {
    const attr = this.getLastAttribute();
    const config = Trix.getBlockConfig(attr ? attr : "default");
    return config ? config.breakOnReturn : false;
  };
  Trix.LineBreakInsertion.prototype.shouldInsertBlockBreak = function () {
    if (this.block.hasAttributes() && this.block.isListItem() && !this.block.isEmpty()) {
      return this.startLocation.offset > 0
    } else {
      return !this.shouldBreakFormattedBlock() ? this.breaksOnReturn : false;
    }
  };
}
let trixChange = () => {
  document.addEventListener("trix-change", setTextToTrix);
}
let copy = () => {
  document.execCommand('selectAll', false, null);
}
let noImg = () => {
  document.addEventListener('trix-file-accept', function (e) {
    e.preventDefault();
  });
}

let openMenu = () => {
  menu.value = !menu.value
}


onMounted(() => {
  noImg();
  trixStyle();
  trixChange();
})

onBeforeUnmount(() => {
  trixChange();

})

</script>

<template>

  <main class="flex-1 w-full h-full mt-12 mb-0 relative">
    <section class="w-full h-auto relative">
    
      <div
        class="space-y-6 flex flex-col lg:mx-12 md:mx-12 mx-0 flex-wrap justify-center items-center relative break-words">
        <div class="flex-1 lg:w-[55rem] md:w-[40rem] sm:w-[35rem] w-[18rem]">
          <span class="lg:text-3xl md:text-3xl text-2xl font-thin block">A Minimal Bionic Reading</span>
          <div class="flex gap-x-3 mt-6">
            <span class="text-lg font-thin block">Inspired by</span>  
            <a class="py-0.5 px-6 bg-slate-600 rounded-md"
              href="https://not-br.neocities.org">
              <span class="text-sm text-white">Not-Br</span>
              </a>
          </div>

          <div class="text-white mt-6">
            <input id="editor" type="hidden" name="content" v-model="trixText">
            <trix-editor @input="convert" @change="convert" input="editor"
              class="bg-slate-200 text-slate-900 leading-normal">

            </trix-editor>
          </div>

        </div>
        <div class="flex-1 lg:w-[55rem] md:w-[40rem] sm:w-[35rem] w-[18rem]">
          <div class="flex justify-end">
            <div class="w-1/2">
              <div class="flex gap-x-3 justify-between">
                <label for="fixation" class="font-semibold">Fixation</label>
                <input v-model="fixation"
                  class="w-16 focus:outline-none text-black border-2 border-slate-300 rounded-md text-center"
                  type="number" max="10" min="0">
  
              </div>
              <input v-model="fixation" @input="convert()" class="w-full" type="range" min="0" max="10" step="1">
  
            </div>

          </div>


        </div>
        <div class="flex-1 lg:w-[55rem] md:w-[40rem] sm:w-[35rem] w-[18rem]">
          <div class="text-center text-slate-900 font-bold mb-3 flex items-center justify-center">
            <span class="text-xl">
              Result
            </span>
          </div>

          <div class="text-white">
            <div contenteditable="true" v-html="result" @cut.prevent="false" @paste.prevent="false"
              @keydown.prevent="(event) => event.metaKey ? true : false" @click="copy"
              class="bg-slate-200 text-slate-900 select-all cursor-default leading-normal focus:outline-none rounded-lg py-1 px-4"
              style="min-height: 15em;">
            </div>
          </div>
        </div>
      </div>
    </section>
  </main>

</template>
