# Having Multiple Blog Layouts

When a client is after having more than one blog - with different styling for each - you can easily split the layouts with the following:

In the `blog.liquid` file, use the blog handle to conditionally pick the layout you're after.
```
{%- if blog.handle == 'news' -%}
  {% section 'blog-template' %}
{%- else -%}
  {% section 'blog-press-template' %}
{%- endif -%}
```