<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  import MarkdownToJson from './MarkdownToJson.svelte';
  import { crossfade } from 'svelte/transition';
  import { cubicInOut } from 'svelte/easing';
  const [send, receive] = crossfade({ duration: 200, easing: cubicInOut });
	
	let activeDiv = 2;
	let showModal = false;

  let colorPalette = ["#FF0000", "#00FF00", "#0000FF", "#FFFF00", "#00FFFF", "#FF00FF", "#C0C0C0", "#808080", "#800000", "#808000"];
  let selectedColor = null;


  let selectedCategory = null;
  let selectedGroup = null;
  let selectedItems = [];
  let reminders = writable([]);
  let phrases = [];

  onMount(async () => {
    try {
      const responseReminders = await fetch('https://raw.githubusercontent.com/editorblitz/bionotes/main/reminders.json');
      if (!responseReminders.ok) {
        throw new Error(`HTTP error! status: ${responseReminders.status}`);
      } else {
        const dataReminders = await responseReminders.json();
        reminders.set(dataReminders);
      }
    } catch (error) {
      console.error("Error fetching reminders:", error);
    }

    try {
      const responsePhrases = await fetch('https://raw.githubusercontent.com/editorblitz/bionotes/main/phrases.json');
      if (!responsePhrases.ok) {
        throw new Error(`HTTP error! status: ${responsePhrases.status}`);
      } else {
        const dataPhrases = await responsePhrases.json();
        phrases = dataPhrases;
      }
    } catch (error) {
      console.error("Error fetching phrases:", error);
    }
  });

  $: console.log($reminders);
  
  let inputText = "";

  function toggleCategory(category, index) {
    if (selectedCategory && selectedCategory.categoryName === category.categoryName) {
      selectedCategory = null;
      selectedGroup = null;
      selectedItems = [];
    } else {
      selectedCategory = category;
      selectedGroup = null;
      selectedItems = [];
    }
    selectedColor = index % colorPalette.length; 
  }

  function toggleGroup(group, index) {
    if (selectedGroup && selectedGroup.group === group.group) {
      selectedGroup = null;
      selectedItems = [];
    } else {
      selectedGroup = group;
      selectedItems = [];
    }
    selectedColor = index % colorPalette.length;
  }


  function toggleItem(item, index) {
  const itemWithColor = { item, color: colorPalette[index % colorPalette.length] };

  if (selectedItems.find(i => i.item.item === item.item)) {
    selectedItems = selectedItems.filter(i => i.item.item !== item.item);
  } else {
    selectedItems = [...selectedItems, itemWithColor];
  }
}


  function createSegments(text) {
    const words = text.split(' ');
    const segments = [];
    words.forEach(word => {
        const foundPhrase = phrases.find(p => p.phrase.toLowerCase() === word.toLowerCase());
        if (foundPhrase) {
            segments.push({
                text: word,
                isMatch: true,
                message: foundPhrase.message
            });
        } else {
            segments.push({
                text: word,
                isMatch: false
            });
        }
    });
    return segments;
  }

  function getMessage(text) {
    const phrase = phrases.find((p) => p.phrase.toLowerCase() === text.toLowerCase());
    return phrase ? phrase.message : null;
  }
    
  const fontAwesomeScript = document.createElement('script');
  fontAwesomeScript.src = 'https://kit.fontawesome.com/a076d05399.js';
  fontAwesomeScript.crossorigin = 'anonymous';
  document.body.appendChild(fontAwesomeScript);
	
</script>

<main>
  
  <button on:click={() => showModal = true}>Markdown Converter</button>

  {#if showModal}
  <div class="modal" out:receive="{{key: 'modal'}}">
    <div class="modal-content" in:send="{{key: 'modal'}}">
        <button on:click="{() => showModal = false}">Close</button>
        <MarkdownToJson/>
    </div>
  </div>
  {/if}

<div class="app-container">
 <div class="button-container">
  <button class="left-panel-button" on:click={() => activeDiv = 1}>
    <i class="fas fa-icon1"></i>
  </button>
  <button class="left-panel-button" on:click={() => activeDiv = 2}>
    <i class="fas fa-icon2"></i>
  </button>
</div>

  <div class="content-container">
    <div style="display: {activeDiv === 1 ? 'block' : 'none'};">
      <!-- The first div content goes here -->
      <div class="column">
        <h3>Enter your text:</h3>
        <textarea bind:value={inputText} />
        <p>
          {#each createSegments(inputText) as { text, isMatch, message }}
            {#if isMatch}
              <span class="highlight">
                {text}
                <span class="tooltip">{message}</span>
              </span>
            {:else}
              {text}
            {/if}
          {/each}
        </p>
      </div>
    </div>

    <div style="display: {activeDiv === 2 ? 'block' : 'none'};">
      <!-- The remaining four divs content goes here -->
			<div class="content-container">
      <div class="column">
        <h3>Broad Area</h3>
        {#each $reminders as category, index}
          <button on:click={() => toggleCategory(category, index)} class={`color-${colorPalette.indexOf(colorPalette[index % colorPalette.length])}`}>{category.categoryName}</button>
        {/each}
      </div>

      <div class="column">
        <h3>Specific</h3>
        {#if selectedCategory}
          {#each selectedCategory.groups as group, index}
            <button class={`color-${colorPalette.indexOf(colorPalette[index % colorPalette.length])}`} on:click={() => toggleGroup(group, index)}>{group.group}</button>
          {/each}
        {/if}
      </div>

      <div class="column">
        <h3>Details</h3>
        {#if selectedGroup}
          {#each selectedGroup.items as item, index}
            <button class={`color-${colorPalette.indexOf(colorPalette[index % colorPalette.length])}`} on:click={() => toggleItem(item, index)}>{item.item}</button>
            <p>{item.description}</p>
          {/each}
        {/if}
      </div>

   <div class="column">
  <h3>Details</h3>
  {#if selectedItems.length}
    {#each selectedItems as {item: {item, details}, color}}
      {#each Object.entries(details) as [key, value]}
        <p class={`color-${colorPalette.indexOf(color)}`}>{key}: {value}</p>
      {/each}
    {/each}
  {/if}
</div>
    </div>
  </div>
</div>
</div>

</main>
	
<style>
  
  .app-container {
    display: flex;
  }
  .button-container {
    display: flex;
    flex-direction: column;
    justify-content: start;
    align-items: start;
  }
  .content-container {
		display: flex; 
    width: 100%;
  }
  .left-panel-button {
    width: 50px;
    height: 50px;
    margin-bottom: 10px;
    border-radius: 0;
  }

  .column {
		flex-basis: 33%;
    background-color: #e0e0e0;
    padding: 16px;
    box-sizing: border-box;
  }

  textarea {
    width: 100%;
    height: 150px;
  }

  .highlight {
    background-color: yellow;
    position: relative;
  }

  .tooltip {
    visibility: hidden;
    width: 120px;
    background-color: black;
    color: #fff;
    text-align: center;
    border-radius: 6px;
    padding: 5px 0;
    position: absolute;
    z-index: 1;
    bottom: 125%; /* Position the tooltip above the text */
    left: 50%;
    margin-left: -60px; /* Use half of the width value to center the tooltip */
    opacity: 0;
    transition: opacity 0.3s;
  }
	
  .highlight:hover .tooltip {
    visibility: visible;
    opacity: 1;
  }
	
.color-0 { color: #FF0000; }
.color-1 { color: #00FF00; }
.color-2 { color: #0000FF; }
.color-3 { color: #FFFF00; }
.color-4 { color: #00FFFF; }
.color-5 { color: #FF00FF; }
.color-6 { color: #C0C0C0; }
.color-7 { color: #808080; }
.color-8 { color: #800000; }
.color-9 { color: #808000; }


	
	</style>
