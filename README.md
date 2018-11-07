We have an app to search and hire developers. The backend, markup and css have been completed but we still need to animate and give some behaviour to the UI with **javascript**.

You can see the final result here :point_right: https://devhire-demo.herokuapp.com/

## Setup

```shell
 git clone git@github.com:rodloboz/devhire.git
 cd devhire
 rails db:create db:migrate db:seed
 rails server
```

Open you browser and visit `localhost:3000`.

## 1 - Animate text with `typed.js`

The landing page, which is rendered by `home#pages`, has a banner with a main heading, a subheading and a search input.

![landing banner](https://github.com/rodloboz/workshops/blob/master/images/01-devhire/Screen%20Shot%202018-11-02%20at%2009.53.35.png?raw=true)

Let's animate the subheading with `typed.js` so that instead of a static sentence, the user will see animated programming language names (`ruby`, `react`, `elixir`, `python`, `swift`) behind the **_devs in the world_**.

**Remember** that no javascript should go in the asset pipeline. We will be coding all javascript in `app/javascript/` and `app/javascript/packs/application.js` is webpack's entry point as required in the `application.html.erb` layout file:

```html
<%= javascript_pack_tag 'application' %>
```

First create a `components` folder inside `app/javascript/`. This is where we will code all of our javascript features.

Create a file `banner.js` inside components. We'll implement the dynamic banner text here. Add `typed.js` to your project using the `yarn` package manager:
```shell
yarn add typed.js
````
Create a function `loadDynamicBannerText` in banner.js.

We have prepared the markup with an element **#js-typed-text** to act as the target for the `Typed` function.

Once you've implemented the feature don't forget to **export** it and then **import** it in `application.js`.

Done? Good! Time to `commit` and `push`.

## 2 - Animate navbar on scroll!

Still on the landing page, our transparent sticky navbar looks strange once you scroll past the banner. Let's update it on scroll!

Create a `navbar.js` in the components folder to implement features related to the navbar.

On the landing page, the navbar starts with the class **`navbar-wagon-transparent`**, which has the properties to make it transparent, and the class **`navbar-wagon-filled`** styles a white navbar.

You can add an __event listener__ to `window` to listen to the `scroll` event. When the window scrolls past halfway of the banner you should __remove__ the `navbar-wagon-transparent` class and __add__ the `navbar-wagon-transparent` class. When you scroll back up, you need to implement the reverse behaviour.

__Hint:__ The window object has a `scrollY` which gives you its y-axis or vertical position and an `innerHeight` that measures how tall the window is.

Don't forget to **export** and **import** your function. Test it and once it's working it's a good time to `commit` and `push`.

## 3 - Autocomplete suggestions with `js-autocomplete`

For the remaining feature of the landing page, we want to have the search input display **suggestions** as the user enters search terms.

A popular implementation of this featue is with Twitter's robust `typeahead.js` and `bloodhound`, but the same effect can be achieved with the simpler `js-autocomplete`.

This library allows  _asyncronous_ requests to fetch data from a remote backend, but when the amount of data is not significant (as in this case), it is better to send all the data to the frontend so that we don't have to send http requests as the user types.

The **id** `#search-data` has been added to the search input element, which receives a stringified json with all the programming languages in the database. Use this selector to read and parse the `dataset-skills`.

You can create the `autocomplete.js` file inside your components folder. To require the library run from the terminal:

```shell
yarn add js-autocomplete
```

You can find this npm package's documentation :point_right: [here](https://goodies.pixabay.com/javascript/auto-complete/demo.html).

Don't forget that this package is shipped with its own **css styles**, so you'll need to import them. You can do this by uncomenting `<%= stylesheet_pack_tag 'application', media: 'all' %>` in `application.html`. Then create a `application.css` file in `app/javascript/packs` and add the following:

```javascript
// app/javascript/packs/application.js
import './application.css';
```

```scss
// app/javascript/packs/application.css
@import 'js-autocomplete/auto-complete.css';
```

When you're done, `commit` and `push`.

## 4 - Bookmarking developers

When users are logged in, they can **bookmark** their favorite developers for future reference. There is a _bookmark icon_ on the top-right corner to each developer card.

![developer card](https://github.com/rodloboz/workshops/blob/master/images/01-devhire/Screen%20Shot%202018-11-05%20at%2014.21.54.png?raw=true)

Right now nothing happens when the user clicks on the icon. Let's change that.

Start by creating a `bookmarking.js` component file. The idea is to add a `click` event listener to each icon which will submit a `post` request to `/bookmarked_developers` to create with the `developer_id` in the body to _create_ a new bookmark **OR** a `delete` request to `/bookmarked_developers/:id` with the `developer_id` in the body to _destroy_ an existing bookmark.

After the server responds, you should **toggle** the respective icon. You can select all icons using the selector `.dev-bookmark i`. Each icon is a _fontawesome_ icon that is **full** (or bookmarked) when it has the class `fas` and is **empty** (or not bookmarked) when it has the class `far`. You should toggle between these two classes.

To communicate with the rails backend, use the `fetch` API. Fetch works with `promisses`, so you can wait for the backend to respond before **toggling** an icon. Here's an example of a fetch request with a promisse:

```javascript
fetch('/posts', {
    method: 'post',
    body: JSON.stringify({content: 'Hello world!'}),
    headers: {
    'Content-Type': 'application/json',
    },
})
    .then(() => console.log('You have just created a post!'))
    .catch(error => console.log('Something went wrong...', error));
```

**Hint**: You can get the `developer_id` from the parent developer card of the icon - they have *ids* in the following format: `id="dev-4"`.

Test the app in the browser and use the rails console `rails c` to confirm if the bookmarks are being sucessfully created and deleted.

## 5 - Adding a Datepicker (`flatpickr`)

Let's move to the developer's individual page: `show#developers`. There's a lot of things missing here, but we'll start with the **booking widget**.

Users should be able to book a developer by picking a _start date_ and an _end date_ date. The backend already has the validation to only accept a booking if **both** dates are present.

Developers have _no unavailable dates_ (they are **always** available). The only **restrictions** are that a user cannot book dates in the past and the _end date_ must be greater than the _start date_.

We'll use `flatpickr` for the calendar picker.

Read the documentation and have a look at the examples here :point_right: https://flatpickr.js.org/

`yarn add flatpickr` to install the npm package and then create a `booking.js` component file and `import flatpickr from 'flatpickr'`.

You will need to create two instances of flatpickr: one for the **start date** and another for the **end date**. The selectors you want to target for each of these are the classes `booking_start_date` and `booking_end_date`.

To create an instance of `flatpikr`, pass the input element via a **selector** and an object `{}` of **options** to `flatpickr`. There are several options available, from specifying the `minDate` and `dateFormat` to providing _callback_ functions on specific **events** like in the example below:

```javascript
flatpickr(inputSelector, {
    minDate: 'today',
    dateFormat: 'd-m-Y',
    onChange: function(selectedDates, selectedDate) {
      // DO SOMETHING
    }
  })
```

**Note:** The _end date input_ should be **disabled** if the _start date input_ is blank.

## 6 - Calculating booking total!

Once the user picks the _start date_ and the _end date_, we should **update** and **display** the total price calculation.

For simplicity, we'll assume that the developer will work **6 hours** on each day they are booked. There is also a **15% service charge** which should be applied to the **subtotal** in order to calculate the **total price**.

Here's what we're going for:

![developer card](https://github.com/rodloboz/workshops/blob/master/images/01-devhire/Screen%20Shot%202018-11-07%20at%2013.40.19.png?raw=true)

On the inital page load, the costs element (`.costs`) is not visible. After the user has picked the _end date_ you should **calculate** and **update** the costs and make this element visible by adding the class `is-visible`.

**Hint:** You might want to use the flatpikr event `onChange`.

Define a function `updateCosts` which received the number of `hours` booked, which is difference in days between the `endDate` and the `startDate` times *6*.

```javascript
let hours = Math.ceil((endDate - startDate) / (1000 * 3600 * 24)) * 6;
```

You can use the following helper function to parse dates from the input fields:

```javascript
const parseDate = (string) => {
  const pattern = /(\d{2})-(\d{2})-(\d{4})/;
  return new Date(string.replace(pattern,'$3-$2-$1'));
}
```

And another helper function to display the price values with commas:

```javascript
const numberWithCommas = (number) => {
  return number.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
}
```

Remember that if the user _clears_ any of the date inputs, you should **hide** the costs section by removing the class `is-visible`.

Play with the [demo](https://devhire-demo.herokuapp.com/) until you get the same behaviour.

Once you're done, `commit` and `push`.

## 7 - Adding a carousel slider (`tiny-slider`)

One last thing to do in the developers page.

At the end of the developers page, there is a portfolio section showcasing the work of each developer. Right now it doesn't look very nice - it's just one long column. Let's turn this section into a carousel **slider**!

![portfolio slider](https://github.com/rodloboz/workshops/blob/master/images/01-devhire/Screen%20Shot%202018-11-07%20at%2016.28.42.png?raw=true)


## 8 - Using a jQuery plugin (`select2`)
TODO: Instructions


## 9 - Adding behaviour to the search subnavbar
TODO: Instructions


## 10 - Filtering results with a range slider (`nouislider`)
TODO: Instructions

