# Modifying checkout V2 and the Plus.func() method

This guide is specific to Checkout V2 and addresses two main areas:

1. The different pages/screens of checkout and how to detect which page you're on with JS.
2. How to deal with changes from your functions being overwritten when the DOM is updated.

If you need more general checkout advice, please refer to this documentation on checkout.liquid available only to Plus merchants and Shopify employees.
If you are looking for the Plus.func code block, it can be found at the end of this guide.

## Checkout pages and steps

The checkout process is composed of a number of different pages and steps. Keep in mind that not all customer checkouts will encounter every page and step listed below.
There are a five types of pages in checkout:

- show : the page template for the various steps of the checkout process
- stock_problems : page shown when there is an inventory issue with any cart items
- processing : page shown while processing payment
- forward : page show when you are taken off-site because of a payment method like PayPal
- thank_you : page shown once order has been processed

The "show" pages are part of the regular checkout process; there are three or four steps in this flow:

- contact_information
- shipping_method
- payment_method
- review : optional step set in the Admin

## Identifying the page and step

The current page and step of checkout is exposed in the Shopify.Checkout object. Using javascript, you can retrieve:

- the current page with Shopify.Checkout.page.
- the current step with Shopify.Checkout.step.
![](https://d2mxuefqeaa7sj.cloudfront.net/s_30D6F3DA55BA3E5A2362DC707BC8A966174293B84334552BC2EF5EF2FB8A64DD_1522293224660_image1.png)

![http://take.ms/QMLu5](images/image1.png)


Example: Check if you are on the contact information step.


    if (Shopify.Checkout.step === 'contact_information') {
      // Do this.
    }

 
Example: Check if you are on the stock problems page, a.k.a. inventory issues page.


    if (Shopify.Checkout.page === 'stock_problems') {
     // Do this.
    }

## More about the Shipping Methods page

At the Shipping Methods step, your functions will need to take into consideration the following situations:

- That shipping methods may not be immediately ready. Some checkouts will have to poll for applicable methods.
- That there may be no applicable shipping methods, and the customer will be told this.

The following block of code has a conditional flow to handle this:


    if (Shopify.Checkout.step === 'shipping_method') {
     if ($('[data-poll-refresh]').length) {
       // polling for applicable shipping methods
    
     } else if ($('[data-shipping-method]').length) {
       // shipping methods are on the screen
    
     }else{
       // not polling anymore, no shipping methods presented
    
     }
    }

When and why is the server polled for shipping methods?
If the merchant is using an API or carrier-calculated rates to offer additional shipping methods, then the Shipping Method page will poll the server and once the polling is complete a page:change event will be triggered.

If the merchant is only using manual shipping methods, that is methods they've set up in the Admin, then the server isnot polled after page load and there is no page:change event.
Even though clicking different shipping methods changes the text in the Order Summary section, this does not trigger the page:change event. This is being accomplished through Javascript; the server is not involved.

## Event listeners and functions

Checkout has two custom events:

1. page:load - fires when the page is initially written.
2. page:change - fires when the page is updated with information requested from the server.

When writing new elements to the DOM, there is a risk that the events that trigger page:change will have re-painted a section you have modified - this means your changes will be gone.
Example: In this video, you can see that a subheading was added below the Contact Information heading. However, apage:change event is triggered when the reduction form is submitted and the DOM is updated removing the modification.
In the example above, you'll see that the page:chage event was triggered multiple times. This will be discussed later.

## Event types

If you need to know when a page:load or page:change event is triggered, this information is accessible in the event's type. Here's a quick block of code to demonstrate.


    $(document).on('page:load page:change', function(e) {
     if ( e.type === 'page:load' ) {
       console.log( 'Loaded page.' );
     } else if ( e.type === 'page:change' ) {
       console.log( 'DOM has been changed.' );
     }
    });

If you want a "vanilla" JS method, the code block will add event listeners for the page:change and page:load events. The attachEvent method has been included to cover Internet Explorer versions 5-8.


    var myEventHandler = function(event) {
     // event.type is now "page:load" or "page:change"
     console.log('event.type = ' + event.type);
    };
    
    if (document.addEventListener) {
     document.addEventListener('page:load', myEventHandler);
     document.addEventListener('page:change', myEventHandler);
    } else if (document.attachEvent) {
     // for Internet Explorer versions 5-8
     document.attachEvent('page:load', myEventHandler);
     document.attachEvent('page:change', myEventHandler);
    }

More about page:change events
The page:change event is triggered whenever the server is hit up for information and the DOM is changed. Changing the DOM with your own functions does not trigger the page:change event as the the server was not involved.

Events that trigger page:change include, but are not limited to, :

- Submitting a discount - regardless whether that discount code is valid.
- Changing your shipping country or province/state/region - the server returns relevant provinces and new payment lines that reflect that region's tax rates.
- Displaying shipping methods from third-party APIs - the server is polled for applicable shipping methods and displaying those methods triggers the page:change event.

Note: Displaying Shopify's shipping methods, those manually set by the merchant in the Admin, does not trigger this event.

## Keeping your changes after page:change

To account for this updating of the DOM, we wrap certain functions in the Plus.func( ) method (link to custom code block. This method takes three parameters:

1. name - a string that will be applied as a class attribute to the section parameter; make it unique.
2. section - a string that will specify which DOM element to apply the name on as a class-attribute.
3. callback - the function to run if the targeted section does not have name in its class-attribute.

In practice, you will fire the Plus.func(name, section, callback) method on most page:change events. The check whether the targeted section element already has name as a class-attribute keeps the callback function from being fired multiple times if it is not needed.

## More about the section parameter

The Plus.func( ) method is already set up to handle some values for section.

- "main" - this will be interpreted as '[data-step]' and used as a jQuery selector. It is the main content areaduring the checkout process.
- "discount_form" - this will be interpreted as '[data-reduction-form]'. Submitting the reduction form triggers this change event.
- "payment_lines" - this will be interpreted as '[data-order-summary-section="payment-lines"]'. This is the list of costs in the order summary.
- "payment_due" - this will be interpreted as '[data-order-summary-section="payment-due"]'. This is the last line in Order Summary; chosen to capture any event that changes/reassess the cart total.

Note about Order Summary : Some page:change events update only the subtotals and totals of Order Summary, other events update these sections as well as the discount field section. See this video for a demonstration.
You are encouraged to add additional conditions to handle different values for section to tailor the method to your needs.

## Examples

Example 1: When on the contact information step, add a subheading after the shipping address heading and delete the input for shipping company.


    $(document).on('page:load page:change', function() {
    
     if (Shopify.Checkout.step === 'contact_information') {
       Plus.func('ce-shipping', 'main', function() {
         $('.section--shipping-address .section__header').after('<p>Shipping subheading</p>');
         $('[for="checkout_shipping_address_company"]').closest('.field.field--optional').remove();
       })
     }
    
    })

Example 2: Add a message to the order summary about taxes. This message will need to appear on every step of checkout and the thank you page.

    $(document).on('page:load page:change', function() {
    
     if (Shopify.Checkout.page('show') || Shopify.Checkout.page('thank_you')) {
       Plus.func('ce-taxes', 'payment_due', function() {
         $('[data-order-summary-section="payment-lines"]').append('<p>Tax rate is based on shipping region.</p>');
       })
     }
    
    })

Example 3: Have a message explaining that discounts do not apply to sale items near the discount field. The message must appear on all pages of checkout.


    $(document).on('page:load page:change', function() {
    
     if (Shopify.Checkout.page === 'show') {
       Plus.func('discount-msg', 'discount_form', function() {
          $('[data-reduction-form="update"]').append('<p>Discounts do not apply to sale items</p>');
       })
     }
    
    })

Example 4 Remove all UPS shipping methods at the shipping method step. You'll need to wait until shipping methods from carriers are presented, and you don't want to run the script if no shipping methods are offered.

    $(document).on('page:load page:change', function() {
     if (Shopify.Checkout.step === 'shipping_method') {
    
       if ($('[data-poll-refresh]').length) {
         // polling for applicable shipping methods
       } else if ($('[data-shipping-method]').length) {
         // shipping methods are on the screen
         Plus.func('no-ups', 'main', function() {
           $('[data-shipping-method^="ups-"]').each(function(){
             $(this).parent().remove();
           });
         });
       } else {
         // not polling anymore, no shipping methods presented
       }
     }
    });

## Plus.func method and jQuery

When modifying checkout, use a version of jQuery lower than 2.0 - since 2.0 and above don't support older browsers, namely IE 6, 7 and 8.

    {{ '//ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js' | script_tag }}

The code block below shows a custom Plus object and the methods you'll want to include in checkout.liquid. It's recommended that you write your function expressions/declarations in a separate snippet just to keepcheckout.liquid uncluttered.

    window.Plus =  window.Plus || {} ;
    
    Plus.func = function(name, section, callback) {
     switch (section) {
       case 'main':
         section = '[data-step]';
         break;
       case 'discount_form':
         section = '[data-reduction-form]';
         break;
       case 'payment_lines':
         section = '[data-order-summary-section="payment-lines"]';
         break;
       case 'payment_due':
         section = '[data-order-summary-section="payment-due"]';
         break;
       default:
         section = section;
     }
    
     // checking for a specific class-attribute, else it's been erased by page:change
     if (!$(section).hasClass(name)) {
       if (typeof(callback) === 'function') {
         $(section).addClass(name);
         callback();
       } else {
         console.warn( 'Plus.func requires a function for the callback parameter' );
       }
     }
    }
