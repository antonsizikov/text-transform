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

  let selectedNotation = $state();
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

  function copyToClipboard() {
    navigator.clipboard.writeText(outputText);
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

  <textarea
    class="textarea textarea-bordered"
    bind:value={inputText}
    placeholder="Enter your text here"
  ></textarea>

  <div class="join">
    {#each cases as _case}
      <input
        class="join-item btn"
        type="radio"
        name="case"
        value={_case.value}
        aria-label={_case.label}
        bind:group={selectedCase}
        onchange={notationsAutoHandler}
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
        onchange={notationsAutoHandler}
        >
    {/each}
  </div>

	<select
    class="select bg-base-200"
		bind:value={selectedNotation}
		onchange={() => (notationsSelectionHandler())}
	>
    <option disabled selected>Notation</option>
		{#each notations as notation}
			<option value={notation.value}>
				{notation.label}
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

    <textarea
      readonly
      class="textarea textarea-bordered"
      value={outputText}
      placeholder="Transformed text will appear here"
    ></textarea>
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
