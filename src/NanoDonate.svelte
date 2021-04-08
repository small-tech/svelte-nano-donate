<script>
  ////////////////////////////////////////////////////////////////////////////////
  //
  // Nano Donation component.
  //
  // Like this? Fund us!
  // https://small-tech.org/fund-us
  //
  // Copyright © 2021-present Aral Balkan (https://ar.al) and
  // Laura Kalbag (https://laurakalbag.com), Small Technology Foundation
  // (https://small-tech.org).
  //
  // License: GPLv3.
  //
  ////////////////////////////////////////////////////////////////////////////////

  import { onMount } from 'svelte'
  import QRious from './lib/QRious.mjs'
  import NanoSpinner from './lib/NanoSpinner.svelte'
  import CurrencyOptions from './lib/CurrencyOptions.svelte'
  import { lighten } from './lib/QuickAndDirtyColour.mjs'

  // Props. Pass these to the component.
  export let address = null
  export let amount = 3        // (optional)
  export let currency = 'eur'  // (optional)
  export let theme = null

  // Mnano = 10³⁰ raw
  const Mnano = 1000000000000000000000000000000

  let qrCodeView
  let qrCode
  let exchangeRates = null
  let paymentLink
  let paymentMessage = ''
  let initialisationError = false
  let prefersReducedMotion = false
  let requestError = null

  let modelReady = false

  onMount (async () => {
    if (address === null) {
      initialisationError = true
      return
    }

    prefersReducedMotion = (window.matchMedia('(prefers-reduced-motion: reduce)')).matches

    if (theme !== null) {
      // Apply any CSS variables that were provided in the style property.
      Object.keys(theme).forEach(cssVariable => {
        document.documentElement.style.setProperty(`--${cssVariable}`, theme[cssVariable])

        // Derive the border colour from the (base) colour unless one has been explicitly specified.
        if (cssVariable === 'colour') {
          document.documentElement.style.setProperty(
            '--border-colour',
            theme['border-colour'] === undefined ? lighten(theme['colour'], 50) : theme['border-colour']
          )
        }
      })
    }

    // Get the current exchange rates for nano at the start.
    await getExchangeRates()
  })

  async function getExchangeRates () {
    requestError = null

    let response
    try {
      response = await fetch('https://api.coingecko.com/api/v3/simple/price?ids=nano&vs_currencies=usd,idr,twd,eur,krw,jpy,rub,cny,aed,ars,aud,bdt,bhd,bmd,brl,cad,chf,clp,czk,dkk,gbp,hkd,huf,ils,inr,kwd,lkr,mmk,mxn,myr,ngn,nok,nzd,php,pkr,pln,sar,sek,sgd,thb,try,uah,vef,vnd,zar,xdr')
      if (!response.ok) {
        Promise.reject(response)
      }
      exchangeRates = (await response.json()).nano
    } catch (error) {
      requestError = error
      console.log(error)
      return
    }

    // Initialise the QRCode (nice and large so it looks good on
    // high-resolution displays).
    qrCode = new QRious({
      element: qrCodeView,
      foreground: 'black',
      background: 'white',
      padding: 40,
      value: '',
      size: 1600
    })

    // Update the view.
    updateModel()
  }

  function updateModel () {
    const priceOfCurrencyInNano = exchangeRates[currency]
    const amountInNANO = amount / priceOfCurrencyInNano
    const amountInRaw = amountInNANO * Mnano

    paymentLink = `nano://${address}?amount=${amountInRaw.toLocaleString('fullwide', { useGrouping: false })}`
    paymentMessage = `Send ${amountInNANO.toFixed(6)} NANO`
    qrCode.value = paymentLink

    modelReady = true
  }
</script>

<section>
  <h2>
    Donate NANO
    <!-- Nano logo -->
    <svg viewBox='0 0 1770.2 780.1'>
      <g>
        <path d='M985.7,390c0,55.6-45.1,100.6-100.6,100.6S784.4,445.6,784.4,390c0-75.5-25.2-100.6-100.6-100.6   S583.2,314.6,583.2,390c0,55.6-45.1,100.6-100.6,100.6S381.9,445.6,381.9,390c0-55.6,45.1-100.6,100.6-100.6   c75.5,0,100.6-25.2,100.6-100.6c0-55.6,45.1-100.6,100.6-100.6s100.6,45.1,100.6,100.6c0,75.5,25.2,100.6,100.6,100.6   C940.7,289.4,985.7,334.5,985.7,390z'/>
        <circle cx='281.2' cy='591.3' r='100.6'/>
        <path d='M1589.6,188.7c0,55.6-45.1,100.6-100.6,100.6c-75.5,0-100.6,25.2-100.6,100.6c0,55.6-45.1,100.6-100.6,100.6   c-75.5,0-100.6,25.2-100.6,100.6c0,55.6-45.1,100.6-100.6,100.6c-55.6,0-100.6-45.1-100.6-100.6c0-55.6,45.1-100.6,100.6-100.6   c75.5,0,100.6-25.2,100.6-100.6c0-55.6,45.1-100.6,100.6-100.6c75.5,0,100.6-25.2,100.6-100.6c0-55.6,45.1-100.6,100.6-100.6   C1544.5,88.1,1589.6,133.2,1589.6,188.7z'/>
      </g>
    </svg>
  </h2>

  <form on:submit|preventDefault class:hidden={initialisationError}>
    <fieldset id='nano-amount'>
      <legend class='visually-hidden'>Donate NANO</legend>
      <input id='amount' type='number' min='1' bind:value={amount} on:input={updateModel}>
      <label class='unselectable visually-hidden' for='amount'>Amount</label>
    </fieldset>

    <fieldset id='currency'>
      <select id='currency' bind:value={currency} on:change={updateModel} on:blur={updateModel}>
        <CurrencyOptions />
      </select>
      <label class='unselectable visually-hidden' for='currency'>Currency</label>
    </fieldset>
  </form>

  <div id='output' class:hidden={initialisationError || requestError}>
    {#if exchangeRates === null && requestError === null}
      <div class='loading' role='alert' aria-live='assertive' class:fade-out={exchangeRates !== null || requestError !== null}>
        <NanoSpinner ballTopLeft='#91bced' ballTopRight='#4A90E2' ballBottomLeft='#123c6e' ballBottomRight='#206cc6' size='100' unit='%' />
        <p class:visually-hidden={!prefersReducedMotion}>Loading currencies…</p>
      </div>
    {/if}
    <p id='send-nano-link' class:fade-in={modelReady}>
      <a href={paymentLink}>{paymentMessage}</a>
    </p>
    <canvas id='qrcode-placeholder' width='1600' height='1600'></canvas>
    <canvas class:fade-in={modelReady} bind:this={qrCodeView}></canvas>
  </div>

  <div id='network-error' class:hidden={!requestError}>
    <canvas id='qrcode-placeholder' width='1600' height='1600'></canvas>
    <h3>Network Error</h3>
    <div role='alert'>
      <p>Sorry, we could not load the exchange rates to calculate the price in NANO.</p>
      <button on:click='{getExchangeRates}'>Try again</button>
    </div>
  </div>

  <div id='initialisation-error' class:hidden={!initialisationError}>
    <h3>Initialisation error</h3>
    <p><strong>Missing property (<code>address</code>): </strong> Please pass your wallet address to the donation component using the <code>address</code> property.</p>
    <p><em>For detailed usage instructions, <a href='https://github.com/small-tech/svelte-nano-donate#readme'>please see the readme</a>.</em></p>
  </div>

  <p><small><span>Exchange rates courtesy of </span><a href='https://www.coingecko.com/en/api'>CoinGecko API</a>. <span>Widget by </span><a href='https://small-tech.org/fund-us'>Small Technology Foundation.</a></small></p>
</section>


<style>
  :root {
    --colour: rgb(48, 67, 73);
    --border-colour:rgb(183,186,188);
    --background-colour:white;
  }

  section {
    /* Give form a max-width and center it when it exceeds that width. */
    margin: 1em auto;
    max-width: 21em;
    text-align: center;
    background: var(--background-colour);
  }

  @supports (display: grid) {
    form {
      display: grid;
      grid-template-columns: 30% 1fr;
      grid-template-areas: 'amount currency';
      grid-gap: 0.5em;
    }

    #nano-amount { grid-column: amount; }
    #currency { grid-column: currency; }
  }

  /* Disable the default fieldset styles (border + spacing). */
  fieldset {
    border: none;
    padding: 0;
    display: contents;
  }

  h2 {
    color: var(--colour);
    margin-bottom: 0.5em;
    font-size: 2em;
  }

  h3 {
    font-size: 1.5em;
    line-height: 1;
  }

  p {
    margin-top: 0.75em;
  }

  code {
    font-size: 1.25em;
  }

  input, select, a {
    color: var(--colour);
  }

  input, select, button, #network-error div {
    background: var(--background-colour);
    border: 0.2em solid var(--border-colour);
    border-radius: 5px;
    box-sizing: border-box;
    padding: 0.5em;
    font-size: 1.25em;
    width: 100%;
  }

  button {
    color: white;
    background: var(--colour);
    border-color: var(--colour);
    border-radius: 2em;
    font-size: 1em;
    margin-left: 1em;
    margin-right: 1em;
    width: auto;
  }

  button:hover {
    border-color: var(--border-colour);
  }

  button:active {
    border-color: var(--border-colour);
    transform: scale(0.95, 0.95);
  }

  /* A placeholder canvas for before the QR code loads. */
  canvas#qrcode-placeholder {
    /* Don’t use if grid is not supported. */
    display: none;
  }

  @supports (display: grid) {
    #output, #network-error {
      display: grid;

      /* One column. */
      grid-template-columns: 100%;

      /* 4em = height of #send-nano-link (including margins). */
      /* 100% = fit height of child canvas. */
      grid-template-rows: 4em 100%;

      /* Make all output elements display in place of each other. */
      grid-template-areas: "link" "output";
    }

    #output #send-nano-link {
      grid-area: link;
    }

    #output > *, #network-error > * {
      grid-area: output;
    }

    #network-error h3 {
      grid-area: link;
    }

    #network-error div {
      grid-area: output;
      display: flex;
      flex-direction: column;
      justify-content: center;
    }


    /* A placeholder canvas to stop interface from jumping when the QR code loads. */
    canvas#qrcode-placeholder {
      /* Grid is supported, so use the placeholder. */
      display: block;
      /* Hide behind QR code and loading spinner if they’re present. */
      z-index: -1;
    }

    .loading {
      display: flex;
      flex-direction: row;
      flex-wrap: nowrap;
      justify-content: center;
      align-content: flex-start;
      align-items: center;
    }
  }

  canvas {
    opacity: 0;
    margin-bottom: 0;
    width: 100%;
  }

  svg {
    width: 1.5em;
  }

  svg path, svg circle {
    fill: var(--colour);
  }

  small {
    color: var(--colour);
    font-size: 1em;
    font-style: italic;
    text-align: center;
  }

  /* Note: we can’t do small:not(a) here as the filter is
     applied to the whole small tag regardless although
     the target specifier is correct. */
  small span {
    filter:grayscale(100)
  }

  /* Courtesy: https://stackoverflow.com/a/2310809 */
  .unselectable {
    -webkit-touch-callout: none;
    -webkit-user-select: none;
    -khtml-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
  }

  .fade-in, .fade-out {
    transition: 1.25s ease all;
  }

  .fade-in {
    opacity: 1 !important;
  }

  .fade-out {
    opacity: 0 !important;
  }

  /* Hide items both visually and from assistive technologies. */
  .hidden {
    display: none !important;
  }

  /* Hide items visually while keeping them visible to assistive technology.
    Use with caution and avoid where the content can be happily shown. */
  .visually-hidden {
    border: 0 !important;
    clip: rect(1px, 1px, 1px, 1px) !important;
    -webkit-clip-path: inset(50%) !important;
    clip-path: inset(50%) !important;
    height: 1px !important;
    overflow: hidden !important;
    padding: 0 !important;
    position: absolute !important;
    width: 1px !important;
    white-space: nowrap !important;
  }

  #initialisation-error {
    border: 0.5em solid red;
    padding-left: 1em;
    padding-right: 1em;
    padding-bottom: 0.75em;
    text-align: left;
    height: 428px;
  }

  #network-error p, #network-error h3 {
    color: #E00000;
  }

  #network-error p {
    font-size: 1em;
  }

  #network-error p:nth-of-type(2) {
    font-style: italic;
    font-size: 0.9em;
    color: var(--colour);
    filter: grayscale()
  }

  #network-error div {
    border-color: red;
    color: var(--colour);
    background-color: var(--background-colour);
  }


  #send-nano-link {
    opacity: 0;
    margin-top: 0.75em;
    margin-bottom: 0.5em;
    padding: 0;
    width: 100%;
    text-align: center;
    font-size: 1.5em;
  }

  /* Dark mode. */
  @media (prefers-color-scheme: dark) {
    section { filter: invert(1) hue-rotate(180deg); }

    /* Re-invert the canvas as the QRCode should never be inverted. */
    canvas { filter: invert(1) hue-rotate(180deg); }
  }

  /* Fix the ugly select box arrow in Firefox. */
  /* Courtesy: https://github.com/twbs/bootstrap/issues/16201#issuecomment-498358474 */
  @supports (-moz-appearance:none) {
    select {
      -moz-appearance:none !important;
      background: transparent url('data:image/gif;base64,R0lGODlhBgAGAKEDAFVVVX9/f9TU1CgmNyH5BAEKAAMALAAAAAAGAAYAAAIODA4hCDKWxlhNvmCnGwUAOw==') right center no-repeat !important;
      background-position: calc(100% - 0.5em) center !important;
    }
  }

  /* Apply the same visual style as we’re using in Firefox to select boxes on WebKit-based browsers. */
  @supports (-webkit-appearance:none) {
    select {
      -webkit-appearance:none !important;
      background: transparent url('data:image/gif;base64,R0lGODlhBgAGAKEDAFVVVX9/f9TU1CgmNyH5BAEKAAMALAAAAAAGAAYAAAIODA4hCDKWxlhNvmCnGwUAOw==') right center no-repeat !important;
      background-position: calc(100% - 0.5em) center !important;
    }
  }
</style>
