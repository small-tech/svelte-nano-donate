<script>
  import { onMount } from 'svelte'
  import QRious from 'qrious'
  import CurrencyOptions from './lib/CurrencyOptions.svelte'
  import { lighten } from './lib/QuickAndDirtyColour.mjs'

  // Props. Pass these to the component.
  export let address
  export let amount = 3        // (optional)
  export let currency = 'eur'  // (optional)
  export let theme = null

  // Mnano = 10³⁰ raw
  const Mnano = 1000000000000000000000000000000

  let qrCodeView
  let qrCode
  let exchangeRates
  let paymentLink
  let paymentMessage = ''

  onMount (async () => {
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
    const response = await fetch('https://api.coingecko.com/api/v3/simple/price?ids=nano&vs_currencies=usd,idr,twd,eur,krw,jpy,rub,cny,aed,ars,aud,bdt,bhd,bmd,brl,cad,chf,clp,czk,dkk,gbp,hkd,huf,ils,inr,kwd,lkr,mmk,mxn,myr,ngn,nok,nzd,php,pkr,pln,sar,sek,sgd,thb,try,uah,vef,vnd,zar,xdr')

    exchangeRates = (await response.json()).nano

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
  })

  function updateModel () {
    const priceOfCurrencyInNano = exchangeRates[currency]
    const amountInNANO = amount / priceOfCurrencyInNano
    const amountInRaw = amountInNANO * Mnano

    paymentLink = `nano://${address}?amount=${amountInRaw.toLocaleString('fullwide', { useGrouping: false })}`
    paymentMessage = `Send ${amountInNANO.toFixed(6)} NANO`
    qrCode.value = paymentLink
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

  <form on:submit|preventDefault>
    <fieldset id='nanoAmount'>
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

  <p id='sendNanoLink'>
    <a href={paymentLink}>{paymentMessage}</a>
  </p>

  <canvas bind:this={qrCodeView}></canvas>

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

  /* Disable the default fieldset styles (border + spacing). */
  fieldset {
    border: none;
    padding: 0;
    display: contents;
  }

  h2 {
    color: var(--colour);
  }

  input, select, a {
    color: var(--colour);
  }

  input, select {
    border-style: solid;
    background: var(--background-colour);
    border: 0.2em solid var(--border-colour);
    border-radius: 5px;
    box-sizing: border-box;
    padding: 0.5em;
    font-size: 1.25em;
    width: 100%;
  }

  canvas {
    margin-bottom: 0.5em;
    max-width: 21em;
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
    display: block;
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

  #sendNanoLink {
    margin-top: 0.5em;
    margin-bottom: 0.5em;
    padding: 0;
    width: 100%;
    text-align: center;
    font-size: 1.5em;
  }

  @supports (display: grid) {
    form {
      display: grid;
      grid-template-columns: 30% 1fr;
      grid-template-areas: 'amount currency';
      grid-gap: 0.5em;
    }

    #nanoAmount { grid-column: amount; }
    #currency { grid-column: currency; }
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

  /* Remove the webkit-specific default styles for the select box too */
  @supports (-webkit-appearance:none) {
    select {
      -webkit-appearance:none !important;
      background: transparent url('data:image/gif;base64,R0lGODlhBgAGAKEDAFVVVX9/f9TU1CgmNyH5BAEKAAMALAAAAAAGAAYAAAIODA4hCDKWxlhNvmCnGwUAOw==') right center no-repeat !important;
      background-position: calc(100% - 0.5em) center !important;
    }
  }

</style>
