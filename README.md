# @small-tech/svelte-nano-donate

A [Nano](https://nano.org) donation component built using Svelte.

⯈ [Live demo](https://small-tech.org/demo/svelte-nano-donate/)

___Note:__ you can [use this component in any web project](#use-in-non-svelte-apps), you __do not__ need to use Svelte yourself._

## Install

```shell
npm i @small-tech/svelte-nano-donate
```

## Use (in Svelte apps)

Simply import and use the component.

All you need to use the component is the Nano address you want to receive donations into:

```html
<script>
  import Donate from '@small-tech/svelte-nano-donate'
</script>

<Donate address=nano_1hfmyyjtm4muiudw1m8dz54jry13jx8xykp9kz1mx3uqe4dtsb3yrdkjgy6g/>
```

You can also customise the component by specifying properties like the initial `amount` and `currency` that’s displayed. And you even specify a theme to customise how the component looks:

```html
<script>
  import Donate from '@small-tech/svelte-nano-donate'
</script>

<main>
  <Donate
    amount=5
    currency=usd
    address=nano_1hfmyyjtm4muiudw1m8dz54jry13jx8xykp9kz1mx3uqe4dtsb3yrdkjgy6g
    theme={{colour: '#4A90E2'}}
  />
</main>
```

Here’s [a live demo of this example built using SvelteKit](https://small-tech.org/demo/svelte-nano-donate) ([source code](https://github.com/small-tech/svelte-nano-donate-demo/)).

The `colour` value specified here will be used for the text. Also, the `border-colour` of form elements will automatically be calculated as a lighter tint of this colour, unless you specify that property explicitly.

The component comes with automatic dark mode support.

For the full list of theme properties, please see the [Theming](#theming) section.

## Use (in non-Svelte apps)

You can `require()` or `import()` the component in your projects as it is simply a regular JavaScript class.

To use it, instantiate the class and pass it an options object that contains the DOM element that you want the class to bind to any properties you want to include (only the `address` property is required). The plain HTML and JavaScript snippet below is functionally equivalent to the Svelte version, above.

```html
<main>
  <div id='nano-donate'></div>

  <script type='module'>
    import NanoDonate from '@small-tech/svelte-nano-donate'

    const nanoDonate = new NanoDonate({
      target: document.getElementById('nano-donate'),
      props: {
        amount: 5,
        currency: 'usd',
        address: 'nano_1hfmyyjtm4muiudw1m8dz54jry13jx8xykp9kz1mx3uqe4dtsb3yrdkjgy6g',
        theme: {colour: '#4A90E2'}
      }
    })
  </script>
</main>
```

This example is included in the _example/_ folder. Run it in a local server (e.g., using [Site.js](https://sitejs.org)) to see it in action.

## Theming

By default, you get a theme with light and dark modes.

You can also apply an entirely custom theme to the component by passing an object to the `theme` property.

The list of valid colour names for a custom `theme` object are shown here and they map to the CSS variables used in the component:

| Name                | Default            | Used for                                          |
| ------------------- | ------------------ | ------------------------------------------------- |
| `colour`            | `rgb(48, 67, 73)`  | Text in form elements and links                   |
| `border-colour`     | `rgb(183,186,188)` | Borders of form elements; 50% lighter than colour |
| `background-colour` | `white`            | Component background                              |

If you don’t explicitly pass a custom `border-colour` value, it will be calculated automatically from the value of the `colour` property. _Note that for this automatic calculation to succeed, you must specify the value of the `colour` property in either rgb(a) or hex format (not as a CSS colour name)._

## Develop

To contribute to the development of this component, please use the discussion feature on GitHub to discuss any changes/improvements you want to make and open issues if you notice any bugs before working on a feature/fix and submitting a pull request.

### Build

```
npm run build
```

This will output an ECMAScript module and a UMD script in the `dist` folder. The build script runs automatically via the `prepublishOnly` hook in package.json when publishing to npm.

## Like this? Fund us!

[Small Technology Foundation](https://small-tech.org) is a tiny, independent not-for-profit.

We exist in part thanks to patronage by people like you. If you share [our vision](https://small-tech.org/about/#small-technology) and want to support our work, please [become a patron or donate to us](https://small-tech.org/fund-us) today and help us continue to exist.

## Copyright

&copy; 2021-present [Aral Balkan](https://ar.al), [Small Technology Foundation](https://small-tech.org).

## License

[ISC](https://opensource.org/licenses/ISC)

