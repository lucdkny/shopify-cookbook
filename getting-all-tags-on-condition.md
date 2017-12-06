### How to return all of the tags in Shopify. 

Go to the shop. Then go to "collections/all"
Add add the code blocks below at the top of the collection template. 

Will get all tags. 

```
{% for tag in collection.all_tags %}
	  {{ tag }},
{% endfor %}
```


All tags that does not include certain words in a tag. 

```
{% for tag in collection.all_tags %}
	{% unless tag contains "color" or tag contains "category" or tag contains "brand"  or tag contains "device" %}
	  {{ tag }},
	{% endunless %}
{% endfor %}
```

All tags that will return any word that contains "device-" tags. 

```
{% for tag in collection.all_tags %}
	{% if tag contains "device-" %}
	  {{ tag }},
	 {% endif %}
{% endfor %}
```