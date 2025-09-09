<script lang="ts">
  let inputText = $state('');
  let selectedCase = $state('skip');
  let selectedSpacing = $state('skip');
  let isClean = $state(false);
  let outputText = $derived(transformText(inputText));

  let notations = $state([
    //'Regular case',
    'flatcase',
    'camelCase', // Fix
    'PascalCase', // Fix
    'kebab-case',
    'snake_case',
    'SCREAMING_SNAKE_CASE'
  ]);
  let selectedNotation = $state();

  function transformCase(inputText: string, selectedCase: string): string {
    switch (selectedCase) {
      case 'lower':
        return inputText.toLowerCase();
      case 'upper':
        return inputText.toUpperCase();
      case 'capital':
        return inputText.charAt(0).toUpperCase() + inputText.slice(1).toLowerCase();
      case 'title':
        return inputText
          .split('\n')
          .map(line =>
            line
              .split(/([ \-_])/)
              .map(part =>
                /[ \-_]/.test(part)
                  ? part 
                  : part.charAt(0).toUpperCase() + part.slice(1).toLowerCase()
              )
              .join('')
          )
          .join('\n');
      case 'invert':
        return inputText.split('').map(char => char === char.toUpperCase() ? char.toLowerCase() : char.toUpperCase()).join('');
      case 'camel':
        return inputText
          .split('\n')
          .map(line => {
            return line
              .split(/[\s\-_]+/)          // разбиваем по пробелам, дефисам и подчёркиваниям
              .filter(Boolean)            // убираем пустые элементы (например при "--")
              .map((word, i) =>
                i === 0
                  ? word.toLowerCase()    // первое слово -> всё маленькими
                  : word.charAt(0).toUpperCase() + word.slice(1).toLowerCase()
              )
              .join('');
          })
          .join('\n');
      case 'skip':
        return inputText;
    }
  }

  function transformSpasing(inputText: string, selectedSpacing: string): string {
    switch (selectedSpacing) {
      case 'space':
        return inputText
          .split('\n')
          .map(line => line.replace(/-|_/g, ' '))
          .join('\n');
      case 'kebab':
        return inputText
          .split('\n')
          .map(line => line.replace(/[\s_]/g, '-'))
          .join('\n');
      case 'snake':
        return inputText
          .split('\n')
          .map(line => line.replace(/[\s-]/g, '_'))
          .join('\n');
      case 'none':
        return inputText
          .split('\n')
          .map(line => line.replace(/[_\s-]+/g, ''))
          .join('\n');
      case 'skip':
        return inputText;
    }
  }

  function cleanText(inputText: string): string {
    return inputText
      .split('\n')
      .filter(line => line.trim().replace(/[_\s-]+/g, '').length > 0) // Filter out empty lines
      .map(line => line
        .replace(/^[\s_\-]+|[\s_\-]+$/g, '') // Remove leading and trailing spaces, dashes and underscores
        .replace(/[\s]+/g, ' ') // Replace multiple spaces with a single space
        .replace(/[-]+/g, '-') // Replace multiple dashes with a single dashe
        .replace(/[_]+/g, '_') // Replace multiple underscores with a single underscore
      )
      .join('\n');
  }

  function transformText(inputText: string): string {
    if (isClean) {
      return cleanText(transformSpasing(transformCase(inputText, selectedCase), selectedSpacing));
    }
    return transformSpasing(transformCase(inputText, selectedCase), selectedSpacing);
  }

  function copyToClipboard() {
    navigator.clipboard.writeText(outputText);
  }

  function notationsHandler1() {
    switch (selectedNotation) {
      case 'flatcase':
        selectedCase = 'lower';
        selectedSpacing = 'none';
        break;
      case 'camelCase':
        selectedCase = 'camel'; // add camel case
        selectedSpacing = 'none';
        break;
      case 'PascalCase':
        selectedCase = 'title';
        selectedSpacing = 'none';
        break;
      case 'kebab-case':
        selectedCase = 'lower';
        selectedSpacing = 'kebab';
        break;
      case 'snake_case':
        selectedCase = 'lower';
        selectedSpacing = 'snake';
        break;
      case 'SCREAMING_SNAKE_CASE':
        selectedCase = 'upper';
        selectedSpacing = 'snake';
        break;
    }
  }

  function notationsHandler2() {
    if (selectedCase == 'lower' && selectedSpacing == 'none') {
      selectedNotation = 'flatcase';
    } else if (selectedCase == 'camel' && selectedSpacing == 'none') {
      selectedNotation = 'camelCase';
    } else if (selectedCase == 'title' && selectedSpacing == 'none') {
      selectedNotation = 'PascalCase';
    } else if (selectedCase == 'lower' && selectedSpacing == 'kebab') {
      selectedNotation = 'kebab-case';
    } else if (selectedCase == 'lower' && selectedSpacing == 'snake') {
      selectedNotation = 'snake_case';
    } else if (selectedCase == 'upper' && selectedSpacing == 'snake') {
      selectedNotation = 'SCREAMING_SNAKE_CASE';
    } else {
      selectedNotation = 'Notation';
    }
  }
</script>

<style global>
  @import 'https://unpkg.com/daisyui@4.12.14/dist/full.css';
  .container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
  }
  textarea {
    resize: none;
    width: 350px;
    height: 150px;
    padding: 8px;
    line-height: 1.2;
    font-family: monospace;
  }
  .join {
    margin: 10px;
  }
  .btn-xs {
    margin: 5px;
  }
  .checkbox {
    margin-right: 5px;
  }
  .select {
    margin: 10px;
  }
</style>

<div class="container mx-auto flex flex-col items-center gap-4">
  <textarea class="textarea textarea-bordered" bind:value={inputText} placeholder="Enter your text here"></textarea>

  <div class="join">
    <input class="join-item btn" type="radio" name="case" value="skip" aria-label="Skip" bind:group={selectedCase} onchange={notationsHandler2}>
    <input class="join-item btn" type="radio" name="case" value="title" aria-label="Title" bind:group={selectedCase} onchange={notationsHandler2}>
    <input class="join-item btn" type="radio" name="case" value="capital" aria-label="Capital" bind:group={selectedCase} onchange={notationsHandler2}>
    <input class="join-item btn" type="radio" name="case" value="lower" aria-label="lower" bind:group={selectedCase} onchange={notationsHandler2}>
    <input class="join-item btn" type="radio" name="case" value="upper" aria-label="UPPER" bind:group={selectedCase} onchange={notationsHandler2}>
    <input class="join-item btn" type="radio" name="case" value="invert" aria-label="iNVERT" bind:group={selectedCase} onchange={notationsHandler2}>
  </div>

  <div class="join">
    <input class="join-item btn" type="radio" name="spacing" value="skip" aria-label="Skip" bind:group={selectedSpacing} onchange={notationsHandler2}>
    <input class="join-item btn" type="radio" name="spacing" value="space" aria-label="Space" bind:group={selectedSpacing} onchange={notationsHandler2}>
    <input class="join-item btn" type="radio" name="spacing" value="kebab" aria-label="Kebab" bind:group={selectedSpacing} onchange={notationsHandler2}>
    <input class="join-item btn" type="radio" name="spacing" value="snake" aria-label="Snake" bind:group={selectedSpacing} onchange={notationsHandler2}>
    <input class="join-item btn" type="radio" name="spacing" value="none" aria-label="None" bind:group={selectedSpacing} onchange={notationsHandler2}>
  </div>

	<select
    class="select bg-base-200"
		bind:value={selectedNotation}
		onchange={() => (notationsHandler1())}
	>
    <option disabled selected>Notation</option>
		{#each notations as notation}
			<option value={notation}>
				{notation}
			</option>
		{/each}
	</select>

  <label class="label">
    <input type="checkbox" class="checkbox" bind:checked={isClean}/>
    Trim spaces and remove dups
  </label>

  <div class="indicator">
    <div class="indicator-item indicator-top">
      <button class="btn btn-xs" onclick={copyToClipboard}>Copy</button>
    </div>
    <textarea readonly class="textarea textarea-bordered" value={outputText} placeholder="Transformed text will appear here"></textarea>
  </div>

</div>

<!-- Text for testing:
One two THREE fOURE
One-two-THREE-FOURE
One_two_THREE_FOURE

    some    text    
----some----text----
____some____text____
-->
