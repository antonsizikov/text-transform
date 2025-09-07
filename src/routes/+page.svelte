<script lang="ts">
  let inputText = $state('');
  let selectedCase = $state('');
  let selectedSpacing = $state('');
  // let selectedNotation = $state('');
  let isClean = $state(false);
  let outputText = $derived(transformText(inputText));

  function returnSpasing(selectedSpacing: string): string {
    switch (selectedSpacing) {
      case 'space':
        return ' ';
      case 'kebab':
        return '-';
      case 'snake':
        return '_';
      case 'none':
        return '';
      default:
        return ' ';
    }
  }

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
          .map(line => line
          .split(/[-_\s]/)
          .map(word => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
          .join(returnSpasing(selectedSpacing)))
          .join('\n');
      case 'invert':
        return inputText.split('').map(char => char === char.toUpperCase() ? char.toLowerCase() : char.toUpperCase()).join('');
      default:
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
      default:
        return inputText;
    }
  }

  function cleanText(inputText: string, cbStrip: boolean): string {
    return inputText
      .split('\n')
      .filter(line => line.trim().replace(/[_\s-]+/g, '').length > 0) // Filter out empty lines
      .map(line => line
        .replace(/^[\s_\-]+|[\s_\-]+$/g, '') // Remove leading and trailing spaces and dashes
        .replace(/[\s_\-]+/g, ' ') // Replace multiple spaces with a single space
      )
      .join('\n');
  }

  function transformText(inputText: string): string {
    if (isClean) {
      return transformCase(transformSpasing(cleanText(inputText, isClean), selectedSpacing), selectedCase);
    }
    return transformCase(transformSpasing(inputText, selectedSpacing), selectedCase);
  }

  function copyToClipboard() {
    navigator.clipboard.writeText(outputText);
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
</style>

<div class="container mx-auto flex flex-col items-center gap-4">
  <textarea class="textarea textarea-bordered" bind:value={inputText} placeholder="Enter your text here"></textarea>

  <div class="join">
    <input class="join-item btn" type="radio" name="case" value="title" aria-label="Title" bind:group={selectedCase}>
    <input class="join-item btn" type="radio" name="case" value="capital" aria-label="Capital" bind:group={selectedCase}>
    <input class="join-item btn" type="radio" name="case" value="lower" aria-label="lower" bind:group={selectedCase}>
    <input class="join-item btn" type="radio" name="case" value="upper" aria-label="UPPER" bind:group={selectedCase}>
    <input class="join-item btn" type="radio" name="case" value="invert" aria-label="iNVERT" bind:group={selectedCase}>
  </div>

  <div class="join">
    <input class="join-item btn" type="radio" name="spacing" value="space" aria-label="Space" bind:group={selectedSpacing}>
    <input class="join-item btn" type="radio" name="spacing" value="kebab" aria-label="Kebab" bind:group={selectedSpacing}>
    <input class="join-item btn" type="radio" name="spacing" value="snake" aria-label="Snake" bind:group={selectedSpacing}>
    <input class="join-item btn" type="radio" name="spacing" value="none" aria-label="No" bind:group={selectedSpacing}>
  </div>

  <label class="label">
    <input type="checkbox" class="checkbox" bind:checked={isClean}/>
    Trim spaces and remove dups
  </label>

  <!-- <select class="select">
    <option disabled selected>Notation</option>
    <option>Regular case</option>
    <option>flatcase</option>
    <option>camelCase</option>
    <option>PascalCase</option>
    <option>kebab-case</option>
    <option>snake_case</option>
    <option>SCREAMING_SNAKE_CASE</option>
  </select> -->

  <!-- <div class="join join-vertical">
    <input class="join-item btn" type="radio" name="notations" value="flatcase" aria-label="flatcase" bind:group={selectedOption}>
    <input class="join-item btn" type="radio" name="notations" value="camelCase" aria-label="camelCase" bind:group={selectedOption}>
    <input class="join-item btn" type="radio" name="notations" value="PascalCase" aria-label="PascalCase" bind:group={selectedOption}>
    <input class="join-item btn" type="radio" name="notations" value="kebab-case" aria-label="kebab-case" bind:group={selectedOption}>
    <input class="join-item btn" type="radio" name="notations" value="snake_case" aria-label="snake_case" bind:group={selectedOption}>
    <input class="join-item btn" type="radio" name="notations" value="SCREAMING_SNAKE_CASE" aria-label="SCREAMING_SNAKE_CASE" bind:group={selectedOption}>
  </div> -->

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
