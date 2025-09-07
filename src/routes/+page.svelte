<script lang="ts">
  let inputText = $state('');
  let selectedOption = $state('');
  let outputText = $derived(transformText(inputText, selectedOption));

  function transformText(inputText: string, selectedOption: string): string {
    switch (selectedOption) {
      case 'lower':
        return inputText.toLowerCase();
      case 'upper':
        return inputText.toUpperCase();
      case 'capital':
        return inputText.charAt(0).toUpperCase() + inputText.slice(1).toLowerCase();
      case 'title':
        return inputText
          .split('\n')
          .map(line => line.split(' ').map(word => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase()).join(' '))
          .join('\n');
      case 'invert':
        return inputText.split('').map(char => char === char.toUpperCase() ? char.toLowerCase() : char.toUpperCase()).join('');
      default:
        return inputText;
    }
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
</style>

<div class="container mx-auto flex flex-col items-center gap-4">
  <textarea class="textarea textarea-bordered" bind:value={inputText} placeholder="Enter your text here"></textarea>

  <div class="join">
    <input class="join-item btn" type="radio" name="case" value="title" aria-label="Title" bind:group={selectedOption}>
    <input class="join-item btn" type="radio" name="case" value="capital" aria-label="Capital" bind:group={selectedOption}>
    <input class="join-item btn" type="radio" name="case" value="lower" aria-label="lower" bind:group={selectedOption}>
    <input class="join-item btn" type="radio" name="case" value="upper" aria-label="UPPER" bind:group={selectedOption}>
    <input class="join-item btn" type="radio" name="case" value="invert" aria-label="iNVERT" bind:group={selectedOption}>
  </div>

  <div class="indicator">
    <div class="indicator-item indicator-top">
      <button class="btn btn-xs" onclick={copyToClipboard}>Copy</button>
    </div>
    <textarea readonly class="textarea textarea-bordered" value={outputText} placeholder="Transformed text will appear here"></textarea>
  </div>
  
  <!-- <div class="join join-vertical">
    <input class="join-item btn" type="radio" name="notations" value="flatcase" aria-label="flatcase" bind:group={selectedOption}>
    <input class="join-item btn" type="radio" name="notations" value="camelCase" aria-label="camelCase" bind:group={selectedOption}>
    <input class="join-item btn" type="radio" name="notations" value="PascalCase" aria-label="PascalCase" bind:group={selectedOption}>
    <input class="join-item btn" type="radio" name="notations" value="kebab-case" aria-label="kebab-case" bind:group={selectedOption}>
    <input class="join-item btn" type="radio" name="notations" value="snake_case" aria-label="snake_case" bind:group={selectedOption}>
    <input class="join-item btn" type="radio" name="notations" value="SCREAMING_SNAKE_CASE" aria-label="SCREAMING_SNAKE_CASE" bind:group={selectedOption}>
  </div> -->


</div>
