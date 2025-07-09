---
# You can also start simply with 'default'
theme: apple-basic
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
---




# Studying coding behaviour in delab

Some results after talking to 13 delab members about how they code


---
layout: default
---

# Motivation

 - I (we) code quite a lot 
 - If I am just 1% better at coding, hopefully my productivity compounds 
 - Start of a postdoc is a good time to start afresh 

---

# Talk outline 

---
layout: section
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
  })
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
  })

})



</script>

<div style="display: flex; justify-content: center; align-items: center; gap: 40px;">
  <div id="plot" style="width: 70%; height: 500px;"></div>
  <div id="plot-language-pie" style="width: 30%; height: 500px;"></div>
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

  Plotly.newPlot('plot-os', traces, {
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
  })
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
  })

})



</script>


<div style="display: flex; justify-content: center; align-items: center; height: 100%;">
  <div id="plot-os" style="width: 50%; height: 500px;"></div>
  <div id="plot-pie" style="width: 30%; height: 500px;"></div>
</div>

---

## Some other numbers


---
layout: section
---


# Part 2: some qualitative results 


---

# What is your coding set up like?


- Everyone uses VsCode except me, who uses PyCharm
- Most people don't use particular extensions with VsCode, except for the language-specific ones

