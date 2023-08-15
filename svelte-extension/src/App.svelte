<script>
	import { onMount } from "svelte";
	import { Home, Settings, Languages, Sun, Moon, Gem, ArrowDownUp} from "lucide-svelte";

	let mainContentVisible = true;
	let settingsPageVisible = false;
	let languagePageVisible = false;
	let languageMessageVisible = false;
	let background = getLocalStorage("backgroundPreference") || "solarized";
	let currency1 = getLocalStorage("currency1") || "USD";
	let currency2 = getLocalStorage("currency2") || "EUR";
	let conValueInput = getLocalStorage("conValueInput") || "";
	let conValue2Input = getLocalStorage("conValue2Input") || "";
	let exchangeRates = {};
	let conversionRate = 1;

	$: updateBodyBackground(background);
	$: setLocalStorage("currency1", currency1);
	$: setLocalStorage("currency2", currency2);
	$: setLocalStorage("conValueInput", conValueInput);
	$: setLocalStorage("conValue2Input", conValue2Input);

	function getLocalStorage(key) {
		try {
			return localStorage.getItem(key);
		} catch (error) {
			console.error(`Failed to get item from local storage: ${error}`);
			return null;
		}
	}

	function setLocalStorage(key, value) {
		try {
			localStorage.setItem(key, value);
		} catch (error) {
			console.error(`Failed to set item in local storage: ${error}`);
		}
	}

	function updateBodyBackground(bg) {
		switch (bg) {
			case "darkMode":
				document.body.style.background =
					"linear-gradient(45deg, rgb(0, 0, 0), rgb(50, 50, 50))";
				document.body.style.color = "white";
				break;
			case "lightMode":
				document.body.style.background =
					"linear-gradient(45deg, rgb(255, 255, 255), rgb(150, 150, 150))";
				document.body.style.color = "black";
				break;
			default:
				document.body.style.background =
					"linear-gradient(45deg, rgb(0, 54, 17), rgb(22, 130, 58))";
				document.body.style.color = "white";
				break;
		}
		setLocalStorage("backgroundPreference", bg);
	}

	function showMainContent() {
		mainContentVisible = true;
		settingsPageVisible = false;
		languagePageVisible = false;
	}

	function showSettingsPage() {
		mainContentVisible = false;
		settingsPageVisible = true;
		languagePageVisible = false;
	}

	function showLanguagePage() {
		mainContentVisible = false;
		settingsPageVisible = false;
		languagePageVisible = true;
	}

	function displayLanguageMessage(event) {
		event.stopPropagation();
		languageMessageVisible = true;
	}

	onMount(async () => {
		await fetchExchangeRates(currency1);
		updateConversion();
	});

	async function fetchExchangeRates(baseCurrency) {
		try {
			const response = await fetch(
				`https://v6.exchangerate-api.com/v6/51e69e5e47ac104caa6a4bd0/latest/${baseCurrency.toUpperCase()}`
			);
			const data = await response.json();
			exchangeRates = data.conversion_rates;
		} catch (error) {
			console.log("");
		}
	}

	function convertCurrency(fromCurrency, toCurrency) {
		if (
			exchangeRates &&
			exchangeRates[fromCurrency.toUpperCase()] &&
			exchangeRates[toCurrency.toUpperCase()]
		) {
			const conversionRate =
				exchangeRates[toCurrency.toUpperCase()] /
				exchangeRates[fromCurrency.toUpperCase()];
			return conversionRate;
		}
		return null;
	}

	$: if (currency1 && currency2) {
		conversionRate = convertCurrency(currency1, currency2);
		if (conValueInput) {
			conValue2Input = parseFloat(
				(conValueInput * conversionRate).toFixed(2)
			);
		} else {
			conValue2Input = "";
		}
	}

	function updateConversion() {
		if (currency1 && currency2) {
			conversionRate = convertCurrency(currency1, currency2);
			if (conValueInput) {
				conValue2Input = conValueInput * conversionRate;
			} else {
				conValue2Input = "";
			}
		}
	}

	function handleNumericInput(event) {
		event.target.value = event.target.value
			.replace(/[^0-9.]/g, "")
			.replace(/(\..*?)\./g, "$1");
	}

	function reverseReverse() {
		[currency1, currency2] = [currency2, currency1];
		[conValueInput, conValue2Input] = [conValue2Input, conValueInput];
	}
</script>

<svelte:head>
	<title>Currency converter</title>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
	<link
		href="https://fonts.googleapis.com/css2?family=Lobster&family=Roboto+Mono:wght@500&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<nav>
	<h1 id="title">Currency Convertor</h1>
	<a><button class="home" on:click={showMainContent}><Home /></button></a>
	<a
		><button class="settings" on:click={showSettingsPage}
			><Settings /></button
		></a
	>
	<a
		><button class="language" on:click={showLanguagePage}
			><Languages /></button
		></a
	>
</nav>

<div
	class="border"
	id="main-content"
	style="display: {mainContentVisible ? 'block' : 'none'};"
>
	<div class="container">
		<div class="input-wrapper">
			<input
				on:input={handleNumericInput}
				type="text"
				class="conValue"
				bind:value={conValueInput}
				placeholder="enter amount"
			/>
			<span class="input-currency">{currency1}</span>
		</div>

		<br />
		<div class="select-container">
			<select bind:value={currency1} name="" id="select1">
				<option value="usd">US Dollar (USD)</option>
				<option value="eur">Euro (EUR)</option>
				<option value="jpy">Japanese Yen (JPY)</option>
				<option value="gbp">British Pound (GBP)</option>
				<option value="aud">Australian Dollar (AUD)</option>
				<option value="cad">Canadian Dollar (CAD)</option>
				<option value="chf">Swiss Franc (CHF)</option>
				<option value="cny">Chinese Yuan (CNY)</option>
				<option value="sek">Swedish Krona (SEK)</option>
				<option value="nzd">New Zealand Dollar (NZD)</option>
			</select>
			<select bind:value={currency2} name="" id="select2">
				<option value="usd">US Dollar (USD)</option>
				<option value="eur">Euro (EUR)</option>
				<option value="jpy">Japanese Yen (JPY)</option>
				<option value="gbp">British Pound (GBP)</option>
				<option value="aud">Australian Dollar (AUD)</option>
				<option value="cad">Canadian Dollar (CAD)</option>
				<option value="chf">Swiss Franc (CHF)</option>
				<option value="cny">Chinese Yuan (CNY)</option>
				<option value="sek">Swedish Krona (SEK)</option>
				<option value="nzd">New Zealand Dollar (NZD)</option>
			</select>
		</div>
		<br /><br />
		<div class="input-wrapper">
			<input
				bind:value={conValue2Input}
				placeholder={currency2}
				disabled="true"
				class="endValue"
			/>
			<span class="input-currency2">{currency2}</span>
		</div>
		<br /><br />
		<button on:click={reverseReverse}><ArrowDownUp /></button>
	</div>
</div>

<div
	id="settings-page"
	class="border"
	style="display: {settingsPageVisible ? 'block' : 'none'};"
>
	<div class="container">
		<h1>Color Theme</h1>
		<div class="scheme">
			<button on:click={() => (background = "darkMode")} class="dark"
				><Moon /></button
			>
			<button on:click={() => (background = "lightMode")} class="white"
				><Sun /></button
			>
			<button on:click={() => (background = "solarized")} class="money"
				><Gem /></button
			>
		</div>
	</div>
</div>

<div
	id="language-page"
	class="border"
	style="display: {languagePageVisible ? 'block' : 'none'};"
>
	<div class="container">
		<h1>Choose language</h1>
		<br />
		<select name="" id="" on:click={displayLanguageMessage}>
			<option value="">english</option>
		</select>
		<br />
		<h1 style="display:{languageMessageVisible ? 'block' : 'none'};">
			we only support english bub
		</h1>
	</div>
</div>

<style>
	.input-currency, .input-currency2{
		font-size: 2em;
	}
	#main-content input {
		font-size: 3em;
	}
	#main-content select {
		font-size: 2em;
	}
	.endValue {
		color: black;
		opacity: 1;
		text-align: center;
		background-color: transparent !important;
		border: none !important;
		font-size: 5em !important;
	}
	.input-wrapper {
		margin-top: 30px;
	}
	.select-container {
		display: flex;
		justify-content: space-between;
		width: 100%;
	}

	#main-content select {
		width: calc(
			50% - 10px
		); /* Reduced 10px for some space between the selectors */
	}
	#main-content .container {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	#main-content .conValue,
	#main-content .endValue {
		width: 100%;
		height: 50px;
		margin-bottom: 10px;
	}

	#main-content #select1,
	#main-content #select2 {
		margin-bottom: 20px;
	}

	h1 {
		text-align: center;
	}
	:global(body) {
		text-align: center;
	}
	.scheme {
		display: flex;
		justify-content: space-between;
	}
	.scheme button:not(:last-child) {
		margin-right: -30px;
	}
	.scheme button {
		width: 100px;
		height: 100px;
		display: flex;
		align-items: center;
		justify-content: center;
		border: none;
		transition: 0.3s;
		cursor: pointer;
	}

	.scheme button:hover {
		transform: scale(1.1);
	}

	.dark {
		background: linear-gradient(45deg, rgb(0, 0, 0), rgb(50, 50, 50));
	}

	.white {
		background: linear-gradient(
			45deg,
			rgb(255, 255, 255),
			rgb(150, 150, 150)
		);
	}

	.money {
		background: linear-gradient(45deg, rgb(0, 54, 17), rgb(22, 130, 58));
	}

	#title {
		font-family: "Lobster", monospace;
		font-size: 2rem;
		margin-right: 25px;
	}
	nav {
		display: flex;
		justify-content: space-between;
		align-items: center;
		max-height: 60px;
	}

	.input-wrapper {
		position: relative;
		display: inline-block;
	}

	.input-wrapper input {
		padding-right: 60px;
		width: 100%;
		box-sizing: border-box;
	}
	.input-wrapper .input-currency {
		position: absolute;
		right: 10px;
		top: 50%;
		z-index: 10;
		color: black;
		transform: translateY(-50%);
	}
	.endValue {
		background-color: white;
		color: black;
		opacity: 1;
	}
	:global(body) {
		font-family: "Roboto Mono", monospace;
		position: relative;
		overflow: hidden;
	}

	.border {
		width: 400px;
		height: 450px;
		background: rgba(255, 255, 255, 0.1);
		border-radius: 10px;
		backdrop-filter: blur(10px);
		box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
		border: 1px solid rgba(255, 255, 255, 0.2);
		position: relative;
	}
</style>
