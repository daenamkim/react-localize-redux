## Do I have to connect every component that needs translations?

No you shouldn't have to connect every component. To avoid this add translate to a parent component, and then pass translations 
down to stateless child components as props. See the [pass multiple translations to components](/features/#pass-multiple-translations-to-components) feature on one way to accomplish this.

---------------

## What if my translation data isn't in the required format?

If you don't have control over the translation data for your application you can use the [translationTransform](/api/action-creators/#initializelanguages-options) option. 
This allows you to write a function that takes in your custom translation data, and outputs the data in the required format.
See [Custom data format](/formatting-translation-data/#custom-data-format) for documentaion.

---------------

## How do I persist active language?

Persisting the user’s selected language  after a refresh can be done a few ways, and how that is done is really up to you. 
The following are two approaches you could use:

**1. Save active language to local storage**

When you start your app check localstorage for an existing saved language. If one exists use that as default language, if not default to first language.

```javascript
const languages = ['en', 'fr', 'es'];
const defaultLanguage = storage.getItem('language') || languages[0];
store.dispatch(initialize(languages, { defaultLanguage }));
```
Whenever you change the active language update the language stored in localstorage.

```javascript
store.dispatch(setActiveLanguage('fr'));
storage.setItem('language', 'fr');
```

**2. Keep active language in the url**

Assuming you're using [react-router](https://reacttraining.com/react-router/web) you can use [url-params](https://reacttraining.com/react-router/web/example/url-params) to track active language. Your url might look something like this, but the key is that it has the `:language` param. Now when you start your app use the url's language param, and set default language based on that.

```javascript
const languages = ['en', 'fr', 'es'];
const defaultLanguage = this.props.match.params.language;
store.dispatch(initialize(languages, { defaultLanguage }));
```

---------------

## How do I retrieve a translation for a language other than active language?

Let's say your app's active language is English, but you want
to display a single translation in French. You can accomplish this by overriding the `translate` function's `defaultLanguage` option. 

```javascript
import { getTranslate } from 'react-localize-redux';

const Greeting = ({ translate }) => (
  <div>
    <!-- This will be English translation since active language is 'en' -->
    <h1>{ translate('greeting') }</h1>
    <!-- Since we specify defaultLanguage: 'fr' translation will be in French -->
    <h1>{ translate('greeting', null, {defaultLanguage: 'fr'}) }</h1>
  </div>
);

const mapStateToProps = state => ({
  translate: getTranslate(state.locale)
});

export default connect(mapStateToProps)(Greeting);
```

---------------

## How do I handle currency, date, and other localization transformations?

This logic is purposely excluded from react-localize-redux to ensure that both package size and API remian small. If you do require this logic you have the choice of writing it yourself, or using a third party library that specializes in that area e.g.([Moment](https://momentjs.com/) for dates).

Here's an example of a basic currency translation using [reselect](https://github.com/reactjs/reselect):

```javascript
import { initialize } from 'react-localize-redux';

const languages = ['en', 'fr'];

const translations = {
  currency: ["$ ${value}", "${value} $"]
};

store.dispatch(initialize(languages));
store.dispatch(addTranslation(translations));

const Transactions = ({translateCurrency}) => (
  <ul>
    <li>{translateCurrency(1000)}</li>     // renders $ 1,000 (en), 1 000 $ (fr)
    <li>{translateCurrency(100000)}</li>   // renders $ 100,000 (en), 100 000 $ (fr)
    <li>{translateCurrency(500)}</li>      // renders $ 500 (en), 500 $ (fr)
  </ul>
);

const getTranslateSelector = (state) => getTranslate(state.locale);
const getActiveLanguageSelector = (state) => getActiveLanguage(state.locale);

/**
 * Returns a function that takes a number, and formats it using
 * the current language, the currency translate data, and native toLocaleString 
 */
const getTranslateCurrency = createSelector(
  getTranslateSelector, getActiveLanguageSelector,
  (translate, language) => (value) => {
    const localizedValue = value.toLocaleString(language.code);
    return translate('currency', {value: localizedValue});
  }
);

const mapStateToProps = state => ({
  translateCurrency: getTranslateCurrency(state)
});

const LocalizedTransactions = connect(mapStateToProps)(Transactions);
```
---------------

## Can I use [ImmutableJS](https://facebook.github.io/immutable-js/)?

If your redux state is an ImmutableJS [Map](https://facebook.github.io/immutable-js/docs/#/Map), and you're using the [localize](/api/higher-order-component) HOC you'll need to use the [getStateSlice](/api/higher-order-component) option. This option allows you to instruct `localize` on how to read the state even though it's an ImmutableJS Map.

```javascript
import React from 'react';
import { toJS } from 'immutable';
import { localize } from 'Localize';

/**
 * The getStateSlice function will passed the entire state as a param.
 * You are responsible for returning the locale state slice.
 */
const getStateSlice = (state) => state.toJS()['locale'];
localize(Component, 'locale', getStateSlice);
```

---------------

## How does react-localize-redux differ from [react-intl](https://github.com/yahoo/react-intl)?

* **react-intl** is larger in size/complexity, and for good reason as it handles many things related to localization. e.g. Pluralization, currency. Where as with **react-localize-redux** you could still do pluralization, and currency, but you'd be writing the formatting functionality yourself. 
<br/>
<br/>
* **react-intl** doesn't work with Redux out of the box, and needs an additional library [react-intl-redux](https://github.com/ratson/react-intl-redux) to add support.
<br/>
<br/>
* For further discussion on this topic see [original github issue](https://github.com/ryandrewjohnson/react-localize-redux/issues/21).

