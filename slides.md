---
# You can also start simply with 'default'
theme: apple-basic
colorSchema: light
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: Studying coding behaviour in delab
info: None
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# open graph
# seoMeta:
#  ogImage: https://cover.sli.dev
layout:  statement
---




# Studying coding behaviour in delab



Some results after talking to 13 delab members about how they code

<script setup>
import { onMounted, ref } from 'vue'

const code = ref('')
const source = `import numpy as npY
import matplotlib.pyplot as MpLPL_t 
import pandas as PANDAS
from sklearn.decomposition import PCA 

fig, ax = MpLPL_t.subplots()
`

onMounted(() => {
  let i = 0
  const loop = () => {
    code.value = source.slice(0, i)
    i = (i + 1) % (source.length + 1)
    setTimeout(loop, 100)
  }
  loop()
})
</script>

<pre><code class="language-python">{{ code }}</code></pre>



---
layout: image-right
image: image1.jpg
---

# Motivation

<v-clicks>

 - I (we) code quite a lot, so hopefully any small improvement compounds
 - Start of a postdoc is a good time to start afresh : no need to work around early PhD code

 </v-clicks>

---
layout: center
---

# Talk outline 

<v-clicks>

- Part 1: Some quantitative results 
  - Programming language 
  - Operating System 
  - Misc coding habits 
- Part 2: Some qualitative results 
  - Coding set up (editors)
  - Code organisation 
  - Useful libraries / extensions

</v-clicks>

---
layout: intro-image-right
image: image2.jpeg
---

# Part 1: some quantitative results


---

## Which programming language do you use?




<script setup lang="ts">
import Plotly from 'plotly.js-dist'
import { onMounted } from 'vue'



onMounted(async () => {
  const response = await fetch('survey-results.csv')
  const text = await response.text()
  const lines = text.trim().split('\n')
  const header = lines[0].split(',').map(h => h.trim())
  const rows = lines.slice(1)

  // Dynamically extract column indices
  const nameIdx = header.indexOf('name')
  const questionIdx = header.indexOf('question')
  const answerIdx = header.indexOf('answer')

  if (nameIdx === -1 || questionIdx === -1 || answerIdx === -1) {
    console.error("Missing one of the required columns: name, question, answer")
    return
  }

  const data = rows.map(line => {
    const cols = line.split(',').map(c => c.trim())
    return {
      name: cols[nameIdx],
      question: cols[questionIdx],
      answer: +cols[answerIdx]
    }
  })

  const languageData = data.filter(d => d.question.startsWith('language-'))
  const languages = [...new Set(languageData.map(d => d.question.replace('language-', '')))]
  const names = [...new Set(languageData.map(d => d.name))]

  console.log("Languages:", languages)
  console.log("Names:", names)
  console.log("Data:", languageData)

  const traces = names.map(name => {
    const personData = languageData.filter(d => d.name === name)
    const answerMap = Object.fromEntries(
      personData.map(d => [d.question.replace('language-', ''), d.answer])
    )
    const yValues = languages.map(lang => answerMap[lang] ?? 0)
    return {
      x: languages,
      y: yValues,
      type: 'scatter',
      mode: 'lines+markers',
      name: name,
      hoverinfo: 'y',
      uid: name  // 👈 gives each trace a unique ID for selection logic
    }
  })

  console.log("Traces:", traces)

  Plotly.newPlot('plot', traces, {
   title: undefined,
   yaxis: {
      title: { text: 'Usage (%)', font: { size: 16, color: '#333' } },
      rangemode: 'tozero',
      automargin: true
    },
    xaxis: {
      title: { text: 'Language', font: { size: 16, color: '#333' }, standoff: 20 },
      type: 'category',
      automargin: true,
    },
    margin: { t: 20, b: 80, l: 80, r: 0 },
    showlegend: false, 
    font: {
    family: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif',
    size: 14,
    color: '#333'
  },
  }, {displayModeBar: false})
  let highlightedName: string | null = null

  const plotDiv = document.getElementById('plot')

  plotDiv.on('plotly_click', (event) => {
    const clickedName = event.points[0].data.name

    if (highlightedName === clickedName) {
      // Reset: show all traces
      Plotly.restyle(plotDiv, { opacity: 1 })
      highlightedName = null
    } else {
      // Dim all, highlight one
      const traces = plotDiv.data.map(trace => trace.name)
      const opacities = traces.map(name => name === clickedName ? 1 : 0.1)
      Plotly.restyle(plotDiv, { opacity: opacities })
      highlightedName = clickedName
    }
  })

  // 🔸 Pie chart: most-used language per person
  const langByPerson = names.map(name => {
    const personData = languageData.filter(d => d.name === name)
    const topLang = personData.reduce((max, curr) =>
      !max || curr.answer > max.answer ? curr : max, null)
    return topLang ? topLang.question.replace('language-', '') : null
  }).filter(Boolean)

  const langCounts = languages.map(lang => ({
    lang,
    count: langByPerson.filter(x => x === lang).length
  })).filter(entry => entry.count > 0)

  const pieData = [{
    type: 'pie',
    labels: langCounts.map(d => d.lang),
    values: langCounts.map(d => d.count),
    textinfo: 'label+percent',
    hoverinfo: 'label+value+percent',
    marker: { line: { color: '#fff', width: 1 } }
  }]

  Plotly.newPlot('plot-language-pie', pieData, {
    margin: { t: 40, b: 40, l: 50, r: 40 },
    showlegend: false,
    font: {
      family: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif',
      size: 14,
      color: '#333'
    },
    paper_bgcolor: 'rgba(0,0,0,0)',
    plot_bgcolor: 'rgba(0,0,0,0)'
  }, {displayModeBar: false})


})



</script>

<div style="display: flex; justify-content: center; align-items: center; gap: 40px;">
  <div id="plot" style="width: 600px; height: 500px;"></div>
  <div id="plot-language-pie" style="width: 400px; height: 400px;"></div>
</div>

---

## Which operating system do you use?


<script setup lang="ts">
import Plotly from 'plotly.js-dist'
import { onMounted } from 'vue'



onMounted(async () => {
  const response = await fetch('survey-results.csv')
  const text = await response.text()
  const lines = text.trim().split('\n')
  const header = lines[0].split(',').map(h => h.trim())
  const rows = lines.slice(1)

  // Dynamically extract column indices
  const nameIdx = header.indexOf('name')
  const questionIdx = header.indexOf('question')
  const answerIdx = header.indexOf('answer')

  if (nameIdx === -1 || questionIdx === -1 || answerIdx === -1) {
    console.error("Missing one of the required columns: name, question, answer")
    return
  }

  const data = rows.map(line => {
    const cols = line.split(',').map(c => c.trim())
    return {
      name: cols[nameIdx],
      question: cols[questionIdx],
      answer: +cols[answerIdx]
    }
  })

  const osData = data.filter(d => d.question.startsWith('os-'))
  const osLabels = [...new Set(osData.map(d => d.question.replace('os-', '')))]
  const names = [...new Set(osData.map(d => d.name))]


  const traces = names.map(name => {
    const personData = osData.filter(d => d.name === name)
    const answerMap = Object.fromEntries(
      personData.map(d => [d.question.replace('os-', ''), d.answer])
    )
    const yValues = osLabels.map(lang => answerMap[lang] ?? 0)
    return {
      x: osLabels,
      y: yValues,
      type: 'scatter',
      mode: 'lines+markers',
      name: name,
      hoverinfo: 'y',
      uid: name  // 👈 gives each trace a unique ID for selection logic
    }
  })

  console.log("Traces:", traces)

  Plotly.newPlot('plot-os', traces, 
  {
   title: undefined,
   yaxis: {
      title: { text: 'Usage (%)', font: { size: 16, color: '#333' } },
      rangemode: 'tozero',
      automargin: true
    },
    xaxis: {
      title: { text: 'Language', font: { size: 16, color: '#333' }, standoff: 20 },
      type: 'category',
      automargin: true,
    },
    margin: { t: 20, b: 80, l: 80, r: 40 },
    showlegend: false, 
    font: {
    family: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif',
    size: 14,
    color: '#333'
  }, 
  },  {displayModeBar: false}
  )
  let highlightedName: string | null = null

  const plotDiv = document.getElementById('plot-os')

  plotDiv.on('plotly_click', (event) => {
    const clickedName = event.points[0].data.name

    if (highlightedName === clickedName) {
      // Reset: show all traces
      Plotly.restyle(plotDiv, { opacity: 1 })
      highlightedName = null
    } else {
      // Dim all, highlight one
      const traces = plotDiv.data.map(trace => trace.name)
      const opacities = traces.map(name => name === clickedName ? 1 : 0.1)
      Plotly.restyle(plotDiv, { opacity: opacities })
      highlightedName = clickedName
    }
  })

   // Pie chart: most-used OS per person
  const osByPerson = names.map(name => {
    const personData = osData.filter(d => d.name === name)
    const topOS = personData.reduce((max, curr) =>
      !max || curr.answer > max.answer ? curr : max, null)
    return topOS ? topOS.question.replace('os-', '') : null
  }).filter(Boolean)

  const osCounts = osLabels.map(os => ({
    os,
    count: osByPerson.filter(x => x === os).length
  })).filter(entry => entry.count > 0)

  const pieData = [{
    type: 'pie',
    labels: osCounts.map(d => d.os),
    values: osCounts.map(d => d.count),
    textinfo: 'label+percent',
    hoverinfo: 'label+value+percent',
    marker: { line: { color: '#fff', width: 1 } }
  }]

  console.log('osCounts:', osCounts)

  Plotly.newPlot('plot-pie', pieData, {
    margin: { t: 40, b: 40, l: 40, r: 40 },
    showlegend: false,
    font: {
      family: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif',
      size: 14,
      color: '#333'
    },
    paper_bgcolor:'rgba(167, 84, 84, 0)',
    plot_bgcolor: 'rgba(0,0,0,0)'
  }, {displayModeBar: false})



})


</script>


<div style="display: flex; justify-content: left; align-items: center;">
  <div id="plot-os" style="width: 500px; height: 500px;"></div>
  <div id="plot-pie" style="width: 400px; height: 400px;"></div>
</div>

---
layout: center
---

## Some other numbers


<script setup lang="ts">
import Plotly from 'plotly.js-dist'
import { onMounted, nextTick } from 'vue'

onMounted(async () => {
  await nextTick()

  const response = await fetch('survey-results.csv')
  const text = await response.text()
  const lines = text.trim().split('\n')
  const header = lines[0].split(',').map(h => h.trim())
  const rows = lines.slice(1)

  const nameIdx = header.indexOf('name')
  const questionIdx = header.indexOf('question')
  const answerIdx = header.indexOf('answer')

  const answerOrder = ['often', 'regularly', 'sometimes', 'rarely', 'no']
  const answerColors = [
    '#d4e6f1',  // light blue (often)
    '#7fb3d5',  // medium blue (regularly)
    '#2980b9',  // darker blue (sometimes)
    '#1f618d',  // even darker blue (rarely)
    '#154360'   // very dark blue (no)
  ]

  const data = rows.map(line => {
    const cols = line.split(',').map(c => c.trim())
    return {
      name: cols[nameIdx],
      question: cols[questionIdx],
      answer: cols[answerIdx]
    }
  })

  

  // Map of question keys → human-readable titles
  const questionsToPlot = {
    comments: 'How frequently do you <br> comment your code?',
    ai: 'How frequently do you use AI?',
    hpc: 'How frequently do you <br> use the HPC?',
    git: 'How frequently do you use Git?'
  }

  for (const [q, title] of Object.entries(questionsToPlot)) {
    const divId = `pie-${q}`

    const relevantAnswers = data
      .filter(d => d.question === q)
      .map(d => d.answer.trim().toLowerCase())

    const answerCounts = Object.entries(
      relevantAnswers.reduce((acc, a) => {
        acc[a] = (acc[a] || 0) + 1
        return acc
      }, {} as Record<string, number>)
    )

    const countsByAnswer = new Map(answerOrder.map(a => [a, 0]))
    relevantAnswers.forEach(a => {
      if (countsByAnswer.has(a)) {
        countsByAnswer.set(a, countsByAnswer.get(a)! + 1)
      }
    })

    const labels = answerOrder
    const values = answerOrder.map(a => countsByAnswer.get(a) || 0)
    const colors = answerColors

    // Filter out categories with value 0
    const filtered = labels
      .map((label, i) => ({ label, value: values[i], color: colors[i] }))
      .filter(d => d.value > 0)

    const filteredLabels = filtered.map(d => d.label)
    const filteredValues = filtered.map(d => d.value)
    const filteredColors = filtered.map(d => d.color)

    console.log(`Answers for ${q}:`, relevantAnswers)
    console.log('Counts before filtering:', countsByAnswer)
    console.log('Filtered labels:', filteredLabels)
    console.log('Filtered values:', filteredValues)
    console.log('Filtered colors:', filteredColors)

    Plotly.newPlot(divId, [{
      type: 'pie',
      labels: filteredLabels,
      values: filteredValues,
      textinfo: 'label+percent',
      hoverinfo: 'label+value+percent',
      marker: { colors: filteredColors, line: {       color: '#fff', width: 1 } }
    }], {
      title: {
        text: title,
        font: { size: 12 },
        pad: { b: 5 }
      },
      margin: { t: 40, b: 10, l: 80, r: 10 },
      showlegend: false,
      font: {
        family: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif',
        size: 12,
        color: '#333'
      },
      paper_bgcolor: 'rgba(0,0,0,0)',
      plot_bgcolor: 'rgba(0,0,0,0)'
    }, {
      displayModeBar: false
    })
  }
})
</script>

<div style="
  display: grid;
  grid-template-columns: repeat(4, auto);
  column-gap: 40px;
">
  <div id="pie-comments" style="width: 200px; height: 250px;"></div>
  <div id="pie-ai" style="width: 200px; height: 250px;"></div>
  <div id="pie-hpc" style="width: 200px; height: 250px;"></div>
  <div id="pie-git" style="width: 200px; height: 250px;"></div>
</div>


---
layout: intro-image-right
image: image3.jpg
---


# Part 2: some qualitative results 


---
layout: center
---

# What is your coding set up like?

<v-clicks>

- Everyone uses VSCode except me, who uses PyCharm 
- Most people don't use particular extensions with VSCode, except for the language-specific ones

But here are some that are mentioned / others that I found....


</v-clicks>


---
transition: fade
---

# Useful VSCode extensions 


<div v-click>

## Rainbow CSV : colours your csv files

![alt text](https://user-images.githubusercontent.com/5349737/190057249-8ec401f6-b76d-4420-a6af-053dd502f8a9.png)

</div>

---
layout: center
transition: fade
---

# Useful VSCode extensions 

## Indent Rainbow and Rainbow brackets

<div class="grid grid-cols-2 gap-4">

![alt text](https://raw.githubusercontent.com/oderwat/vscode-indent-rainbow/master/assets/example.png)

![alt text](https://github.com/mazesec/vscode-rainbow-brackets/blob/main/images/vue.png?raw=true)

</div>

---
transition: fade
---

# Useful VSCode extensions

## PyLance : autocomplete, navigation and other utils 

![Pylance Features](https://raw.githubusercontent.com/microsoft/pylance-release/main/images/all-features.gif)



---
layout: center
transition: fade
---

# More useful VSCode extensions 

 - Path Intellisense: autocomplete filenames
 - autoDocString: generates python docstrings
 - RemoteExplorer: SSH to remote machines
 - Code Runner: provide shortcuts to evaluate current line / current script in python
 - Error lens: highlights syntax / other simple errors in code 
 - Better comments: colouring system for comments (eg. TODOs, warnings, questions)
 - vscode-spotify



--- 


# How do you structure your (analysis) code?

<v-clicks>

 - most people start with jupyter notebooks, then move to scripts
 - others mainly use jupyter notebooks
 - During the start of my PhD, Maxime Rio from Tom's lab recommended CookieCutter Datascience to me, maybe you will also find that useful

</v-clicks>

---

# Cookiecutter datascience structure

<script setup>
import { ref } from 'vue'

const treeData = [
  { label: '📄 LICENSE' },
  { label: '📄 Makefile' },
  { label: '📄 README.md' },
  {
    label: '📁 data',
    children: [
      { label: '📁 external ← Data from third party sources' },
      { label: '📁 interim ← Intermediate transformed data' },
      { label: '📁 processed ← Final canonical data sets' },
      { label: '📁 raw ← Original immutable data dump' }
    ]
  },
  { label: '📁 docs ← mkdocs project' },
  { label: '📁 models ← Trained models and summaries' },
  { label: '📁 notebooks ← Jupyter notebooks with naming convention' },
  { label: '📄 pyproject.toml ← Project config & tool settings' },
  { label: '📁 references ← Manuals, dictionaries, etc.' },
  {
    label: '📁 reports',
    children: [
      { label: '📁 figures ← Generated graphics for reporting' }
    ]
  },
  { label: '📄 requirements.txt ← Reproducible environment' },
  { label: '📄 setup.cfg ← flake8 config' },
  {
    label: '📁 {{ cookiecutter.module_name }} ← Project source code',
    children: [
      { label: '📄 __init__.py ← Python module init' },
      { label: '📄 config.py ← Configuration variables' },
      { label: '📄 dataset.py ← Data loading/generation' },
      { label: '📄 features.py ← Feature engineering' },
      {
        label: '📁 modeling',
        children: [
          { label: '📄 __init__.py' },
          { label: '📄 predict.py ← Inference code' },
          { label: '📄 train.py ← Model training' }
        ]
      },
      { label: '📄 plots.py ← Visualization code' }
    ]
  }
]

const openStates = ref(treeData.map(() => false))

const toggle = (i) => {
  openStates.value[i] = !openStates.value[i]
}
</script>

<div class="text-left font-mono text-sm">
  <ul>
    <li v-for="(node, i) in treeData" :key="i">
      <div @click="toggle(i)" class="cursor-pointer hover:text-blue-500">
        {{ node.label }}
      </div>
      <ul v-if="openStates[i] && node.children" class="pl-4">
        <li v-for="child in node.children" :key="child.label">
          {{ child.label }}
        </li>
      </ul>
    </li>
  </ul>
</div>


---
layout: center
---

# Useful (Python) libraries 

 - Polars : Fast pandas dataframe
 - pymer4 : R-style linear-mixed effects model in python
 - JAX : for custom neural networks / automatic differentiation
 - Dask : lazy-loading of arrays that don't fit into memory
 - Typer / Rich : Features for bulding command line interface 
 - Decord : efficient loading of videos 



---



# The end 

<v-clicks>


 - Thank you to everyone who I interviewed 
 - Slides were made using sli.dev, they are in https://timothysit.github.io/delab-retreat-2025-presentation

 Some other things I didn't have time to discuss: 

  - writing tests for analysis code
  - python environment management (why are more conda environments so hard to reproduce?)


 </v-clicks>