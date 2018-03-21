# Variant Changing in Slate

If you're looking to change the variant details not via the select dropdown. You can do it by firing a function in `variants.js`.

Here's an example where we needed to change the variant via clicking a slide thumbnail that was tied to a variant image:

```
// Tie this product object to this
var product_this = this;

/*  On click of slide thumbnail, check if it's tied to a variant, then if so,
 *  remove selected from element and pass object to variant function.
 */
this.$thumbs.on('click'+ this.namespace, function(e) {
  e.preventDefault();

  if ($(this).is('[data-product-single-thumbnail]')) {

    var selectedVariant = $(this).data('variantId');
    var selectVariant = document.querySelectorAll('#SingleOptionSelector-1 option');

    if(selectVariant) {
      $.each(selectVariant, function(i, option) {
        if(selectedVariant !== '') {
          option.removeAttribute('selected');
        }
        if(option.getAttribute('data-variant-title') === selectedVariant) {
          option.setAttribute('selected', 'selected');
          product_this.variants._onSelectChange();
        }
      });
    }
    // $('#SingleOptionSelector-1').val(selectedVariant);
    console.log("Product", product_this);
  }
});
```