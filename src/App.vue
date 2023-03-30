<template>
  <div class="app">
    <input
        class="add-file"
        type="file"
        @change="addFile($event)"
        placeholder="Upload file"
        accept=".csv,application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel"
    />
    <div v-if="!loading" class="content">
      <input type="text" v-model="findText" placeholder="Введите текст поиска.." v-show="fileList.length > 0" class="search">
      <ul v-for="(item,idx) in filteredFileList" :key="idx" class="list">
        <hr/>
        <li>
          <p>#{{ idx + 1 }}&nbsp;</p>
          {{ Object.values(item).join(' ') }}
        </li>
      </ul>
      <div style="text-align: center; color: #FF3D00;" v-show="filteredFileList.length === 0">Нет результатов поиска...
        <hr style="border: 1px solid  #FF3D00; width: 50%;"/>
      </div>
    </div>
    <div v-else style="text-align: center;"><span class="loader"></span></div>


  </div>

</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import * as XLSX from 'xlsx'

const arrayBuffer = ref()
const fileList = ref([])
const file = ref(null)
const loading = ref(false)
const findText = ref('')

onMounted(() => {
if(localStorage.getItem('resultArray')) {
  fileList.value = JSON.parse(localStorage.getItem('resultArray'))
}
})
const addFile = (e) => {
  loading.value = true
  fileList.value = []
  localStorage.removeItem('resultArray')
  file.value = e.target.files[0]
  const fileReader = new FileReader()
  fileReader.readAsArrayBuffer(file.value)

  fileReader.onload = () => {
    arrayBuffer.value = fileReader.result
    const data = new Uint8Array(arrayBuffer.value)
    const arr = []
    for (let i = 0; i !== data.length; ++i)
      arr[i] = String.fromCharCode(data[i])
      const bStr = arr.join('')
      const workbook = XLSX.read(bStr, { type: 'binary' })
      const first_sheet_name = workbook.SheetNames[0]
      const worksheet = workbook.Sheets[first_sheet_name]
      fileList.value = XLSX.utils.sheet_to_json(worksheet, { raw: true })
      localStorage.setItem('resultArray', JSON.stringify(fileList.value))
    loading.value = false
  }
}

const filteredFileList = computed(() => {
  if (!findText.value) {
    return fileList.value
  }
  return fileList.value.filter((item) => {
    if (
        Object.values(item).join(' ').toLowerCase().includes(findText.value.toLowerCase())
    ) {
      return Object.values(item).join(' ')
    }
  })
})
// const filteredFileList = computed(() => {
//  return  fileList.value.filter((item) => {
//   Object.values(item).join(' ').toLowerCase().includes(findText.value.toLowerCase())
//   })
// })
console.log(filteredFileList.value)
</script>

<style>
.app {
  padding: .5rem;
  margin: 0 auto;
  width: 100%;
  height: 100%;
}
.add-file {
margin: .5rem;
}
.content {
  width: 100%;
  height: 100%;
}

.search {
  width: 50%;
 padding: .5rem;
  margin: 1rem;
}
li {
  list-style: none;
}
.loader {
  position: relative;
  width: 48px;
  height: 48px;
}
.loader:before,
.loader:after {
  content:"";
  display: block;
  border: 32px solid transparent;
  border-top-color: #676363;
  position: absolute;
  left: 0;
  top: 0;
  animation: weld-rotate 2s infinite ease-in;
}
.loader:before {
  border-color: transparent  transparent transparent #FF3D00;
  animation-delay: 0.5s;
}
@keyframes weld-rotate {
  0% , 25% {transform: rotate(0deg)}
  50% , 75% {transform: rotate(180deg)}
  100% {transform: rotate(360deg)}
}

</style>
