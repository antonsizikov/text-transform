<script lang="ts">
  let inputText = $state('');
  let outputText = $derived(transformText(inputText));
  let isClean = $state(false);

  let selectedCase = $state('skip');
  let cases = $state([
    { value: 'skip', label: 'Skip' },
    { value: 'lower', label: 'lower' },
    { value: 'upper', label: 'UPPER' },
    { value: 'capital', label: 'Capital' },
    { value: 'title', label: 'Title' },
    { value: 'invert', label: 'iNVERT' },
    //{ value: 'camel', label: 'camel' }
  ])

  let selectedSpacing = $state('skip');
  let spacings = $state([
    { value: 'skip', label: 'Skip' },
    { value: 'space', label: 'Space' },
    { value: 'kebab', label: 'Kebab' }, // hyphens
    { value: 'snake', label: 'Snake' }, // underscore
    { value: 'none', label: 'None' }
  ])

  let selectedNotation = $state('Notation');
  let notations = $state([
    { value: 'flatCase', label: 'flatcase' },
    { value: 'camelCase', label: 'camelCase' },
    { value: 'pascalCase', label: 'PascalCase' },
    { value: 'kebabCase', label: 'kebab-case' },
    { value: 'snakeCase', label: 'snake_case' },
    { value: 'screamingSnakeCase', label: 'SCREAMING_SNAKE_CASE' }
  ]);

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
              .split(/[\s\-_]+/)
              .filter(Boolean)
              .map((word, i) =>
                i === 0
                  ? word.toLowerCase()
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

  function notationsSelectionHandler() {
    switch (selectedNotation) {
      case 'flatCase':
        selectedCase = 'lower';
        selectedSpacing = 'none';
        break;
      case 'camelCase':
        selectedCase = 'camel';
        selectedSpacing = 'none';
        break;
      case 'pascalCase':
        selectedCase = 'title';
        selectedSpacing = 'none';
        break;
      case 'kebabCase':
        selectedCase = 'lower';
        selectedSpacing = 'kebab';
        break;
      case 'snakeCase':
        selectedCase = 'lower';
        selectedSpacing = 'snake';
        break;
      case 'screamingSnakeCase':
        selectedCase = 'upper';
        selectedSpacing = 'snake';
        break;
    }
  }

  function notationsAutoHandler() {
    if (selectedCase == 'lower' && selectedSpacing == 'none') {
      selectedNotation = 'flatCase';
    } else if (selectedCase == 'camel' && selectedSpacing == 'none') {
      selectedNotation = 'camelCase';
    } else if (selectedCase == 'Title' && selectedSpacing == 'none') {
      selectedNotation = 'pascalCase';
    } else if (selectedCase == 'lower' && selectedSpacing == 'kebab') {
      selectedNotation = 'kebabCase';
    } else if (selectedCase == 'lower' && selectedSpacing == 'snake') {
      selectedNotation = 'snakeCase';
    } else if (selectedCase == 'upper' && selectedSpacing == 'snake') {
      selectedNotation = 'screamingSnakeCase';
    } else {
      selectedNotation = 'Notation';
    }
  }

  function copyToClipboard() {
    navigator.clipboard.writeText(outputText);
  }

  function saveToStorage() {
    localStorage.text = inputText;
    localStorage.case = selectedCase;
    localStorage.spacing = selectedSpacing;
    localStorage.notation = selectedNotation;
    localStorage.clean = JSON.stringify(isClean);
  }

  function restoreFromStorage() {
    if (localStorage.text) inputText = localStorage.text;
    if (localStorage.case) selectedCase = localStorage.case;
    if (localStorage.spacing) selectedSpacing = localStorage.spacing;
    if (localStorage.notation) selectedNotation = localStorage.notation;
    if (localStorage.clean) isClean = JSON.parse(localStorage.clean);
  }

  function clearStorage() {
    delete localStorage.text;
    delete localStorage.case;
    delete localStorage.spacing;
    delete localStorage.notation;
    delete localStorage.clean;
  }

  function resetAll() {
    inputText = '';
    selectedCase = 'skip';
    selectedSpacing = 'skip';
    selectedNotation = 'Notation';
    isClean = false;
    clearStorage();
  }

  restoreFromStorage();
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
  .fab {
    position: fixed;
    top: 7px;
    right: 7px;
  }
</style>

<div class="container mx-auto flex flex-col items-center gap-4">

  <div class="indicator">
    <textarea
      class="textarea textarea-bordered"
      bind:value={inputText}
      placeholder="Enter your text here"
      onchange={() => {saveToStorage();}}
    ></textarea>
    <div class="indicator-item indicator-top">
      <button class="btn btn-xs" onclick={() => {inputText = ""; saveToStorage();}}>Clear</button>
    </div>
  </div>

  <div class="join">
    {#each cases as _case}
      <input
        class="join-item btn"
        type="radio"
        name="case"
        value={_case.value}
        aria-label={_case.label}
        bind:group={selectedCase}
        onchange={() => {notationsAutoHandler(); saveToStorage();}}
        >
    {/each}
  </div>

  <div class="join">
    {#each spacings as spacing}
      <input
        class="join-item btn"
        type="radio"
        name="spacing"
        value={spacing.value}
        aria-label={spacing.label}
        bind:group={selectedSpacing}
        onchange={() => {notationsAutoHandler(); saveToStorage();}}
        >
    {/each}
  </div>

	<select
    class="select bg-base-200"
		bind:value={selectedNotation}
		onchange={() => {notationsSelectionHandler(); saveToStorage();}}
	>
    <option disabled selected>Notation</option>
		{#each notations as notation}
			<option value={notation.value}>
				{notation.label}
			</option>
		{/each}
	</select>

  <label class="label">
    <input
      type="checkbox"
      class="checkbox"
      bind:checked={isClean}
      onchange={() => {saveToStorage();}}
      />
    Trim spaces and remove dups
  </label>

  <div class="indicator">
    <textarea
      readonly
      class="textarea textarea-bordered"
      value={outputText}
      placeholder="Transformed text will appear here"
    ></textarea>
    <div class="indicator-item indicator-top">
      <button class="btn btn-xs" onclick={copyToClipboard}>Copy</button>
    </div>
  </div>

  <div class="fab">
    <button class="btn btn-sm btn-circle" aria-label="Reset all" onclick={resetAll}>
      <svg
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 -960 960 960"
        height="20px"
        width="20px"
        fill="currentColor"
        style="opacity:0.5"
      ><path d="M480-40q-81 0-152.5-31T203-155q-53-53-84-124.5T88-432q0-22 15.5-37.5T141-485q22 0 37.5 15.5T194-432q0 119 83.5 202.5T480-146q119 0 202.5-83.5T766-432q0-119-83.5-202.5T480-718l26 26q14 14 15 34.5T508-621q-14 17-36.5 18T433-618L315-733q-16-16-16.5-37.5T314-808l114-114q16-16 37.5-15.5T503-921q15 16 15.5 37T503-847l-23 23q81 0 152.5 31T757-709q53 53 84 124.5T872-432q0 81-31 152.5T757-155q-53 53-124.5 84T480-40Z"/></svg>
    </button>
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
