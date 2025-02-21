<script>
	import '$lib/styles/global.css';
	import '@fontsource/fira-code';
	import '@fontsource/open-sans';

	/**
	 * Returns the number as a padded hex string
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
		return number < 0 ? ((number % divisor) + divisor) % divisor : number % divisor;
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
		const chroma = cMax - cMin;

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

		return `hsl(${hue.toFixed(0)}deg ${(100 * saturation).toFixed(0)}% ${(100 * lightness).toFixed(
			0,
		)}%)`;
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
	 * Return linear-light or gamma expanded equivalent of 8-bit sinlge R, G or B input
	 * @param {number} rgb8Bit - r, g or b 8-bit value (e.g. 225)
	 * @returns {number} - the calculated linear-light value
	 */
	function linearLight(rgb8Bit) {
		const sRgb = rgb8Bit / 255;
		return sRgb <= 0.03928 ? sRgb / 12.92 : Math.pow(sRgb + 0.055 / 1.055, 2.4);
	}

	/**
	 * Returns a light or dark text class used to increase contrast between background and text
	 * @param {{red: number, green: number; blue: number}} inputColour - rgb values 0 to 255
	 * @returns{'text-light'|'text-dark'} - returns a class
	 */
	function textColourClass({ red, green, blue }) {
		const colourRelativeLuminance =
			0.2126 * linearLight(red) + 0.7152 * linearLight(green) + 0.0722 * linearLight(blue);
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
					<span class="colour-code">
						{#if colourSystem === 'hex'}
							{hex}
						{:else if colourSystem === 'rgb'}
							rgb({red} {green} {blue})
						{:else}
							{hsl}
						{/if}</span
					>
				</article>
			{/each}
		</div>
	</div>
</main>

<style>
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
