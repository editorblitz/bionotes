<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  import MarkdownToJson from './MarkdownToJson.svelte';
	// saving the modal code below in case want to add back
  //import { crossfade } from 'svelte/transition';
 // import { cubicInOut } from 'svelte/easing';
 // const [send, receive] = crossfade({ duration: 200, easing: cubicInOut });
	
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
    
</script>




	
<main>
  	
<div class="app-container">
 <div class="button-container">
  <button class="left-panel-button" on:click={() => activeDiv = 2}>
    </button>
  <button class="left-panel-button" on:click={() => activeDiv = 1}>
    </button>
	 <button class="left-panel-button" on:click={() => activeDiv = 3}>
    </button>
	 
	 
</div>

  <div class="content-container-overall">
    <div style="display: {activeDiv === 1 ? 'block' : 'none'};">
      <!-- The first div content goes here -->
      <div class="content-container-textpad">
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
        <h3>Items</h3>
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
        <div class={`detail-block color-${colorPalette.indexOf(color)}`}>
          <p><span class="key-text">{key}:</span> {value}</p>
        </div>
      {/each}
    {/each}
  {/if}
</div>
    </div>
  </div>

	
	<div style="display: {activeDiv === 3 ? 'block' : 'none'};">
	   <MarkdownToJson/>
  </div>
</div>		
		
</div>

</main>

<style>
	.body {
		width: 100%;
		flex-grow: 1;
	}
	
  .app-container {
    display: flex;
		flex-grow: 1;
    width: 100%;
}
	
	button {
   border-radius: 8px;  /* Change this to the desired radius */
}
  .button-container {
    display: flex;
    flex-direction: column;
    justify-content: start;
    align-items: start;
}
  
	.content-container-overall {
		display: flex; 
    flex-grow: 1;
		width: 100%;
		height: 100%;
  }
	
	.content-container-textpad {
    width: 100%;
		flex-grow: 1;
	}
	
	
	.content-container {
    display: grid;
    grid-template-columns: repeat(2, 1fr) 2fr 5fr;
    width: 100%;
    height: 100vh;
    gap: 10px;
  }
	
  .left-panel-button {
    width: 50px;
    height: 50px;
    margin-bottom: 10px;
    border-radius: 0;
    padding: 20px;
    box-sizing: border-box; 
}

  .column {
  
  background-color: #e0e0e0;
		flex-grow: 1;
  padding: 16px;
  box-sizing: border-box;
	height: 100%
}

  textarea {
    width: 100%;
    height: 150px;
		flex-grow: 1;
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
	
.color-0 { border: 2px solid #FF0000; }
.color-1 { border: 2px solid #00FF00; }
.color-2 { border: 2px solid #0000FF; }
.color-3 { border: 2px solid #FFFF00; }
.color-4 { border: 2px solid #00FFFF; }
.color-5 { border: 2px solid #FF00FF; }
.color-6 { border: 2px solid #C0C0C0; }
.color-7 { border: 2px solid #808080; }
.color-8 { border: 2px solid #800000; }
.color-9 { border: 2px solid #808000; }

.detail-block {
  border-style: solid;
  border-width: 2px;
  margin-bottom: 10px;
  padding: 5px;
	background-color: #F4F4F4;
	border-radius: 8px;  /* Change this to the desired radius */
}
	
	.key-text {
  font-weight: 600;
}
	
</style>
