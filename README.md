# Task One

The Special Item should be the only item which shows in this category - create a feature that will show the product's second image when it is hovered on.
#######################################################################################
This was implemented by doing a loop of the images found within the product, whatsoever, I had to limit it to 2 images only and then made sure the second image was to be shown on hovering by making sure the `index` was greater than `0`. This is the code used:

```handlebars
{{#each (limit images 2)}}
	{{#if @index ">" 0}}
		<img
			src="{{getImage
				this
				'productgallery_size'
				(cdn theme_settings.default_image_product)
			}}"
			class="card-image"
			alt="{{this.alt}}"
			title="{{this.alt}}"
		/>
	{{/if}}
{{/each}}
```

The code above is found on the components/products/card.html file starting on line 99 (Just below the responsive-image component).
I did style it as well; the css is on the top of the card.html. I usually put the css externally in its own css document but since this was a test, I did not mind to put it here.

# Task Two

Add a button at the top of the category page labeled Add All To Cart. When clicked, the product will be added to the cart. Notify the user that the product has been added.
If the cart has an item in it - show a button next to the Add All To Cart button which says Remove All Items. When clicked it should clear the cart and notify the user.
Both buttons should utilize the Storefront API for completion.
#######################################################################################
As requested, two buttons have been implemented and each one calls their corresponding functions, `createCart()` and `deleteCart()`. Here it is the code for the buttons:

NOTE: I had to inspect(F12) the buttons via the customize theme page in the admin dashboard to find the proper class to use.

```html
<button onclick="createCart()" class="button button--primary" type="button">
	Add
</button>
<button onclick="deleteCart()" class="button button--primary" type="button">
	Remove
</button>
```

Now the funny part is the JS section, the requirements were to have said buttons on the top of the category page, which made me think that I need to grab all(array) the product IDs found on the webpage so I could add them to the cart in a single click. It was a funny task(not going to lie); I will explain the work in the code:

###### Create cart

```js
// I started by creating an empty array that will contain our items.
let itemsArrayInCurrentPage = [];

// In order to grab the IDs, I needed to find something in common in the divs of the products and this one caught my interest
// and as soon as I found out that the values matched the  IDs from admin dashboard I went ahead and used it.
const itemIds = document.querySelectorAll("[data-entity-id]");

// Once I had the divs, I went and created a loop to iterate through the elements and find the ids which were found within the
// dataset object.
itemIds.forEach((item) => {
	itemsArrayInCurrentPage.push(item.dataset.entityId);
});

/*
 * By this point we should have all  the IDs needed(and stored in memory) if we console.log(itemsArrayInCurrentPage) it.
 */

const createCart = async () => {
	// We run through them and return the ids.
	const lineItems = itemsArrayInCurrentPage.map((id) => ({
		quantity: 1, // I assumed having 0 would not work and Docs said the quantity param was required for the API endpoint also.
		productId: id, // This is the ID
	}));

	const options = {
		method: "POST",
		headers: {
			"Content-Type": "application/json",
		},
		body: JSON.stringify({ lineItems, locale: "en" }), // Just making sure to JSON format it
	};

	// We make a POST request to my store endpoint
	await fetch(
		`https://demostore-h10.mybigcommerce.com/api/storefront/carts`,
		options
	)
		.then((response) => response.json())
		.then((response) => {
			// I throw an alert to the user's device and tell them the products have been added to the cart
			alert("Items added to cart");

			// I store the recently created cartId in local storage in order to be used when reloading the page and
			// deleting the corresponding cart
			localStorage.setItem("cartId", response.id);

			// Reload the page
			location.reload();
		})
		.catch((err) => console.error(err));
};
```

###### Delete cart

```js
// This is the reason, I decided to store the id after creating a cart.
let cartId = localStorage.getItem("cartId"); // This is the first thing that came up to my head to delete the correct cart

const deleteCart = async () => {
	const options = {
		method: "DELETE",
		headers: {
			"Content-Type": "application/json",
		},
	};

	// We pass the cartId variable as a parameter in the HTTP request and then proceed to the next step, the data returned.
	await fetch(
		`https://demostore-h10.mybigcommerce.com/api/storefront/carts/${cartId}`,
		options
	)
		.then((response) => response.json())
		.then((response) => {
			// I let the user know his/her car has been cleared out
			alert("Cart has been cleared");
			//  I delete the cartId from local storage to avoid incongruencies when creating new ones
			localStorage.removeItem("cartId");
			// Reload  the page
			location.reload();
		})
		.catch((err) => console.error(err));
};
```

###### Create and Delete cart (full code)

```js
let cartId = localStorage.getItem("cartId");
let itemsArrayInCurrentPage = [];

const itemIds = document.querySelectorAll("[data-entity-id]");

itemIds.forEach((item) => {
	itemsArrayInCurrentPage.push(item.dataset.entityId);
});

const createCart = async () => {
	const lineItems = itemsArrayInCurrentPage.map((id) => ({
		quantity: 1,
		productId: id,
	}));

	const options = {
		method: "POST",
		headers: {
			"Content-Type": "application/json",
		},
		body: JSON.stringify({ lineItems, locale: "en" }),
	};

	await fetch(
		`https://demostore-h10.mybigcommerce.com/api/storefront/carts`,
		options
	)
		.then((response) => response.json())
		.then((response) => {
			alert("Items added to cart");
			localStorage.setItem("cartId", response.id);
			location.reload();
		})
		.catch((err) => console.error(err));
};

const deleteCart = async () => {
	const options = {
		method: "DELETE",
		headers: {
			"Content-Type": "application/json",
		},
	};

	await fetch(
		`https://demostore-h10.mybigcommerce.com/api/storefront/carts/${cartId}`,
		options
	)
		.then((response) => response.json())
		.then((response) => {
			alert("Cart has been cleared");
			localStorage.removeItem("cartId");
			location.reload();
		})
		.catch((err) => console.error(err));
};
```

# Task Three (Bonus)

If a customer is logged in - at the top of the category page show a banner that shows some customer details (i.e. name, email, phone, etc). This should utilize the data that is rendered via Handlebars on the Customer Object.
#######################################################################################

We go back to the category page on line 51. We display the component only if there's an active/loggedIn user.

```handlebars
{{#if customer}}
    {{> components/account/account-banner}}
{{/if}}
```

and this is the account-banner component with some css as well:

```handlebars
<style>
	.myBanner { border-radius: 5px; border: 1px solid #444444; padding: 3rem;
	margin-bottom: 1.5rem !important; } .myBanner-container { width: 100%;
	padding-top: 3rem!important; padding-bottom: 3rem!important; } .myBanner-title
	{ font-size: 3rem; font-weight: 700; } .myBanner-description { font-size:
	1.5rem; }
</style>
<div class="myBanner">
	<div class="myBanner-container">
		<h1 class="myBanner-title">{{customer.name}}</h1>
		<p class="myBanner-description">{{customer.email}}</p>
		<p class="myBanner-description">{{customer.phone}}</p>
	</div>
</div>
```
