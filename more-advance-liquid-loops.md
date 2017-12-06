# More Advance Liquid Loops

Code example from a non-sectioned theme.
Refer to HB.

```liquid
{% comment %}Creates the loop for the reminders{% endcomment %}
{% for i in (1...10) %}

    {% comment %}Creates an add to cart button{% endcomment %}
    {% assign cartHasProductType = false %}
    {% assign cartHasProductTag = false %}

    {% comment %}Captures the settings from the schema{% endcomment %}
    {% capture cart_reminder_enabled %}cart_reminder_enabled{{ i }}{% endcapture %}
    {% capture cart_reminder_product_type %}cart_reminder_product_type{{ i }}{% endcapture %}
    {% capture cart_reminder_product_tag %}cart_reminder_product_tag{{ i }}{% endcapture %}

    {% capture cart_reminder_image %}cart_reminder_image{{ i }}{% endcapture %}
    {% capture cart_reminder_image_mobile %}cart_reminder_image_mobile{{ i }}{% endcapture %}

    {% capture cart_reminder_image_link_type %}cart_reminder_image_link_type{{ i }}{% endcapture %}
    {% capture cart_reminder_image_link %}cart_reminder_image_link{{ i }}{% endcapture %}
    {% capture cart_reminder_image_link_add_to_cart %}cart_reminder_image_link_add_to_cart{{ i }}{% endcapture %}
    {% capture cart_reminder_image_link_add_to_cart_capture %}{{ settings.[cart_reminder_image_link_add_to_cart] }}{% endcapture %}

    {% capture cart_reminder_button_type %}cart_reminder_button_type{{ i }}{% endcapture %}
    {% capture cart_reminder_button_product_link %}cart_reminder_button_product_link{{ i }}{% endcapture %}
    {% capture cart_reminder_button_product_link_capture %}{{ settings.[cart_reminder_button_product_link] }}{% endcapture %}


    {% capture cart_reminder_custom_button_type %}cart_reminder_custom_button_type{{ i }}{% endcapture %}
    {% capture cart_reminder_custom_button_link %}cart_reminder_custom_button_link{{ i }}{% endcapture %}
    {% capture cart_reminder_custom_button_text %}cart_reminder_custom_button_text{{ i }}{% endcapture %}

    {% capture cart_reminder_image_offer %}cart_reminder_image_offer{{ i }}{% endcapture %}
    {% capture cart_reminder_custom_button_offer %}cart_reminder_custom_button_offer{{ i }}{% endcapture %}
    {%- comment %}
    	Cart offer Javascript is located at the bottom of shoo.js.liquid
    {% endcomment -%}

    {% if settings.[cart_reminder_enabled] %}

     {% comment %}Assigns the product type as a variable {% endcomment %}
     {% assign productType = settings.[cart_reminder_product_type] | split: ',' %}
     {% assign productTag = settings.[cart_reminder_product_tag] | split: ','  %}

      {% comment %}Tests if the cart has the product tags{% endcomment %}
      {% for item in cart.items %}
        {% for tag in item.product.tags %}
          {% for tagSetting in productTag %}
            {% if tag == tagSetting %}
              {% assign cartHasProductTag = true %}
            {% endif %}
          {% endfor %}
        {% endfor %}
      {% endfor %}

      {% comment %}Tests if the cart has the product type{% endcomment %}
      {% for item in cart.items %}
        {% for typeSetting in productType %}
          {% if item.product.type == typeSetting %}
            {% assign cartHasProductType = true %}
          {% endif %}
        {% endfor %}
      {% endfor %}


      {% comment %}If the cart has the product type - the image and link will render{% endcomment %}
       {% if cartHasProductType == true or cartHasProductTag == true %}

        <div class="cart-reminder__container{{ i }}">
          {% if settings.[cart_reminder_image_link_type] == 'cart-offer' %}

              <a href="" class="{{ settings.[cart_reminder_image_offer] }}">
                <div class="cart-reminder__image">
                   {{ settings.[cart_reminder_image] | img_url: 'original' | img_tag }}
                </div>
                <div class="cart-reminder__image-mobile">
                  {{ settings.[cart_reminder_image_mobile] | img_url: 'original' | img_tag }}
               </div>
              </a>

         {% elsif settings.[cart_reminder_image_link_type] == 'link' or settings.[cart_reminder_image_link_type] == 'add-to-cart' %}<a href="{% if settings.[cart_reminder_image_link_type] == 'link' %}{{ settings.[cart_reminder_image_link] }}{% elsif settings.[cart_reminder_image_link_type] == 'add-to-cart' %}/cart/add?id={{ all_products[cart_reminder_image_link_add_to_cart_capture].first_available_variant.id }}{% endif %}">{% endif %}

              {% if settings.[cart_reminder_image_link_type] != 'cart-offer' %}

             <div class="cart-reminder__image">
                {{ settings.[cart_reminder_image] | img_url: 'original' | img_tag }}
             </div>
              <div class="cart-reminder__image-mobile">
             {{ settings.[cart_reminder_image_mobile] | img_url: 'original' | img_tag }}
             </div>

             {% endif %}


         {% if settings.[cart_reminder_image_link_type] == 'link' or settings.[cart_reminder_image_link_type] == 'add-to-cart' %}</a>{% endif %}

      {% comment %}Creates an add to cart button{% endcomment %}
       {% if settings.[cart_reminder_button_type] == 'show' %}
         <div class="cart-reminder__add-to-cart">
           <a href="/cart/add?id={{ all_products[cart_reminder_button_product_link_capture].first_available_variant.id }}" class="btn uppercase btn--medium">Add to my bag</a>
         </div>
       {% endif %}

        {% comment %}Creates an add to custom button - with options to go to any collection or page{% endcomment %}
       {% if settings.[cart_reminder_custom_button_type] == "show" %}
         <div class="cart-reminder__add-to-cart">
           <a href="{% if settings.[cart_reminder_custom_button_offer] == 'cart-promo-latex' %}{% else %}{{ settings.[cart_reminder_custom_button_link] }}{% endif %}" class="{% if settings.[cart_reminder_custom_button_offer] == 'cart-promo-latex' %}{{ settings.[cart_reminder_custom_button_offer] }}{% endif %} btn uppercase btn--medium">{{ settings.[cart_reminder_custom_button_text] }}</a>
         </div>
       {% endif %}

        {% comment %}Closes off the conatiner if the cart has the product type{% endcomment %}
       {% if cartHasProductType or cartHasProductTag %}
       </div>
       {% endif %}

       {% endif %}

    {% endif %}

{% endfor %}
```
