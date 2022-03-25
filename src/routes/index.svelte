<script>
  import '@fontsource/fira-code';
  import '@fontsource/open-sans';

  /**
   * Returns a the number as a padded hex string
   * @param {number} number - decimal number to convert to hex (should be 0 to 255)
   * @returns {string} - zero padded string
   */
  function decimalToHex(number) {
    return number.toString(16).padStart(2, '0');
  }

  /**
   * Returns an RGB code for the input colour
   * @param {{red: number, green: number; blue: number}} inputColour - rgb values 0 to 255
   * @returns {string}
   */
  function rgbToHex({ red, green, blue }) {
    return `#${decimalToHex(red)}${decimalToHex(green)}${decimalToHex(blue)}`;
  }

  /**
   * Returns mathematically expected modulus - different to JS definition for negative numbers
   * @param number {number} - number to apply divisor to
   * @param divisor {number}
   * @returns number - modulus of input number
   */
  function modulus(number, divisor) {
    return ((number % divisor) + divisor) % divisor;
  }

  /**
   * Returns an HSL equivalent code for the input colour
   * @param {{red: number, green: number; blue: number}} inputColour - rgb value 0 to 255
   * @returns {string} - CSS style hsl string
   */
  function rgbToHsl({ red, green, blue }) {
    const r = red / 255;
    const g = green / 255;
    const b = blue / 255;

    const cMax = Math.max(r, g, b);
    const cMin = Math.min(r, g, b);
    const chroma = Math.max(cMax - cMin);

    let hue = 0;
    if (chroma > 1e-6) {
      if (cMax === r) {
        hue = 60 * modulus((g - b) / chroma, 6);
      } else if (cMax === g) {
        hue = 60 * ((b - r) / chroma + 2);
      } else {
        hue = 60 * ((r - g) / chroma + 4);
      }
    }
    const lightness = (cMax + cMin) / 2;
    const saturation =
      lightness === 0 || lightness === 1 ? 0 : chroma / (1 - Math.abs(2 * lightness - 1));

    return `hsl(${hue.toFixed(0)}deg ${(100 * saturation).toFixed(0)}\% ${(100 * lightness).toFixed(
      0,
    )}\%)`;
  }

  /**
   * Returns a name for the colour
   * @param {string} colourHex - hex value for colour e.g. #ff9f1c
   * @returns {Promise<string>} - name of colour
   */
  async function colourName(colourHex) {
    const response = await fetch(`https://www.thecolorapi.com/id?hex=${colourHex.slice(1)}`);
    const { name } = await response.json();
    return name.value;
  }

  /**
   * Returns a light or dark text class used to increase contrast between background and text
   * @param {{red: number, green: number; blue: number}} inputColour - rgb values 0 to 255
   * @returns{'text-light'|'text-dark'} - returns a class
   */
  function textColourClass({ red, green, blue }) {
    const r = red / 255;
    const g = green / 255;
    const b = blue / 255;
    const r1 = r <= 0.03928 ? r / 12.92 : Math.pow(r + 0.055 / 1.055, 2.4);
    const g1 = g <= 0.03928 ? g / 12.92 : Math.pow(g + 0.055 / 1.055, 2.4);
    const b1 = b <= 0.03928 ? b / 12.92 : Math.pow(b + 0.055 / 1.055, 2.4);
    const colourRelativeLuminance = 0.2126 * r1 + 0.7152 * g1 + 0.0722 * b1;
    const whiteContrastRatio = 1.05 / (colourRelativeLuminance + 0.05);
    const blackContrastRatio = (colourRelativeLuminance + 0.05) / 0.05;
    return blackContrastRatio > whiteContrastRatio ? 'text-dark' : 'text-light';
  }

  let colours = [
    { red: 0, green: 5, blue: 1 },
    { red: 247, green: 244, blue: 243 },
    { red: 255, green: 159, blue: 28 },
    { red: 48, green: 131, blue: 220 },
    { red: 186, green: 27, blue: 29 },
  ];
  let colourCount = colours.length;

  let colourSystem = 'hex';
</script>

<svelte:head>
  <title>Svelte each demo</title>
  <meta name="description" content="Demo of each " />
</svelte:head>

<header class="header-container">
  <h1>Svelte <code>#each</code>: <code>@const</code> and destructuring</h1>
</header>
<main class="main-container">
  <div class="main-content">
    <h2>Colour Palette</h2>
    <div class="picker">
      <label>
        <input type="radio" bind:group={colourSystem} name="Hex" value={'hex'} />
        Hex</label
      >
      <label>
        <input type="radio" bind:group={colourSystem} name="HSL" value={'hsl'} />
        HSL</label
      >
      <label>
        <input type="radio" bind:group={colourSystem} name="RGB" value={'rgb'} />
        RGB
      </label>
    </div>
    <div class="colours">
      {#each colours as { red, green, blue }, index}
        {@const hex = rgbToHex({ red, green, blue })}
        {@const hsl = rgbToHsl({ red, green, blue })}
        {@const colourClass = textColourClass({ red, green, blue })}
        <article
          style:background-color={hex}
          aria-posinset={index + 1}
          aria-setsize={colourCount}
          class={`colour ${colourClass}`}
        >
          {#await colourName(hex)}
            ...
          {:then name}
            {`${name}: `}
          {/await}
          {#if colourSystem === 'hex'}
            <span class="colour-code">{hex}</span>
          {:else if colourSystem === 'rgb'}
            <span class="colour-code">rgb({red} {green} {blue})</span>
          {:else}
            <span class="colour-code">{hsl}</span>
          {/if}
        </article>
      {/each}
    </div>
  </div>
</main>

<style>
  :global(html) {
    background-color: var(--colour-alt);
    font-family: var(--font-body);
    accent-color: var(--colour-theme);
  }

  :global(body) {
    margin: 0;
    font-weight: var(--font-weight-normal);
  }

  :global(h1, h2) {
    font-family: var(--font-heading);
  }

  :global(h1) {
    font-size: var(--font-size-6);
    font-weight: var(--font-weight-extrabold);
  }

  :global(h2) {
    font-size: var(--font-size-5);
    font-weight: var(--font-weight-bold);
  }

  :global(code) {
    font-family: Fira Code;
  }

  :global(:root) {
    --colour-theme: hsl(35 100% 55%); /* orange peel */
    --colour-brand: hsl(211 71% 53%); /* french blue */
    --colour-alt: hsl(0 0% 87%); /* gainsboro */
    --colour-light: hsl(77 100% 97%); /* ivory */
    --colour-dark: hsl(0 0% 20%); /* jet */

    --spacing-1: 0.25rem;
    --spacing-2: 0.5rem;
    --spacing-4: 1rem;
    --spacing-6: 1.5rem;
    --spacing-12: 3rem;
    --spacing-18: 4.5rem;
    --max-width-wrapper: 48rem;

    --font-size-root: 16px;
    --font-size-1: 1rem;
    --font-size-2: 1.25rem;
    --font-size-3: 1.563rem;
    --font-size-5: 2.441rem;
    --font-size-6: 3.052rem;

    --font-weight-normal: 400;
    --font-weight-bold: 700;
    --font-weight-extrabold: 800;

    --font-heading: 'Open Sans';
    --font-body: 'Open Sans';

    /* CREDIT: https://www.joshwcomeau.com/shadow-palette/ */
    --shadow-color: 0deg 6% 60%;
    --shadow-elevation-medium: -1px 1px 1.4px hsl(var(--shadow-color) / 0.51),
      -2.7px 2.7px 3.7px -1.2px hsl(var(--shadow-color) / 0.43),
      -7.6px 7.6px 10.5px -2.3px hsl(var(--shadow-color) / 0.36),
      -20px 20px 27.6px -3.5px hsl(var(--shadow-color) / 0.29);
  }

  .header-container {
    display: grid;
    background-color: var(--colour-theme);
    border-bottom: var(--spacing-1) solid var(--colour-brand);
    box-shadow: var(--shadow-elevation-medium);
    color: var(--colour-dark);
    height: var(--spacing-24);
    place-content: center;
  }

  .main-container {
    background-color: var(--colour-dark);
    box-shadow: var(--shadow-elevation-medium);
    border-radius: var(--spacing-4);
    color: var(--colour-light);
    width: min(100% - var(--spacing-12), var(--max-width-wrapper));
    margin: var(--spacing-18) auto;
    padding: var(--spacing-12);
    font-size: var(--font-size-3);
  }

  .main-content {
    margin: var(--spacing-6);
    padding: var(--spacing-2) var(--spacing-12) var(--spacing-6);
    background-color: var(--colour-light);
    border-radius: var(--spacing-1);
    color: var(--colour-dark);
  }
  .picker {
    display: flex;
    justify-content: space-around;
    background-color: var(--colour-dark);
    box-shadow: var(--shadow-elevation-medium);
    color: var(--colour-theme);
    padding: var(--spacing-6);
    border-radius: var(--spacing-2);
    font-size: var(--font-size-3);
  }
  .colours {
    margin: var(--spacing-12) auto;
  }
  .colour {
    text-align: center;
    padding: var(--spacing-6);
    place-content: center;
    border-radius: var(--spacing-1);
  }
  .colour-code {
    font-weight: var(--font-weight-bold);
  }
  .text-light {
    color: var(--colour-light);
  }
  .text-dark {
    color: var(--colour-dark);
  }
</style>
