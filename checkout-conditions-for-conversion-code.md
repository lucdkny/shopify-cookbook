# Adding checkout Liquid conditions for conversion code to run only once.

Can use this condition in the **Additional Scripts** in the checkout settings in the store admin.

```
{% if first_time_accessed %}
  <!-- Conversion scripts you want to run only once -->
{% endif %}
```
