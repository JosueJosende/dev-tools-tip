# Animar la altura mediante la propiedad height

```css
  article {
    height: 150px;
    overflow: hidden;
    transition: height .5s;
    interpolate-size: allow-keywords;

    &:hover {
      height: auto;
    }
  }
```

[Ver ejemplo](https://codepen.io/jmjela/pen/gOVwPXe)