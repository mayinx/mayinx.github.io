---
title: Aligning Elements in Bootstrap
tags: ["Bootstrap", "CSS"]
excerpt: How to right-align text in an inline-block element? How to align flex-items using auto-margins? This post gives a quick overview regarding available alignment options supported by Bootstrap.
date: 2019-06-14
---

## Aligning Elements in Bootstrap

Aligning content inside a container like text etc. using the BS-Framework is no rocket science - it's all in the docs after all. However, flipping through the docs to find the right solution for your situation among the options available can be a bit annoying.

The following attempts to ease that process a bit by giving a quick overview regarding the available options to align content inside a container. ...  depending on the nature of the container in question - i.e. if it's a block or inline-block element or if flexbox shall be used etc.:

---------------------------------------------

### Aligning content inside block elements (like `p` etc.)

Use one of the following BS-CSS-classes on your block element:  

- `text-start`, `text-end`, `text-center`
- + responsive class-variations of the above (i.e. `text-md-start`, `text-xl-end` etc.)    

```
<p class="text-start">Start aligned text on all viewport sizes.</p>
<p class="text-center">Center aligned text on all viewport sizes.</p>
<p class="text-end">End aligned text on all viewport sizes.</p> 
```
Docs:
- https://getbootstrap.com/docs/5.0/utilities/text/#text-alignment

---------------------------------------------

### Aligning content inside inline-elements (like columns etc.)  

- `text-right` 

```
<div class="row">
    <div class="col-md-6">Total cost</div>
    <div class="col-md-6 text-right">$42</div>
</div>
```

---------------------------------------------

### Aligning content using Flexbox

### `justify-content`: Changing the alignment of flex items on the flex main axis (per default - i.e. `flex-direction: row` - that's the alignment on the x-axis - so horizontal alignment. Once you change the flex-direction to column, the content is justified along the y-axis - so vertical alignment) 

- `.justify-content-start`
- `.justify-content-end`
- `.justify-content-center`
- `.justify-content-between`
- `.justify-content-around`
- `.justify-content-evenly`
- + responsive class-variations of the above (i.e. `justify-content-md-start`, `justify-content-xl-end` etc.)    


```
<div class="d-flex justify-content-end bd-highlight mb-3">
  <div class="p-2 bd-highlight">Flex item</div>
  <div class="p-2 bd-highlight">Flex item</div>
  <div class="p-2 bd-highlight">Flex item</div>
</div>
```

BS-Docs on `justify-content`: 
- https://getbootstrap.com/docs/5.0/utilities/flex/#justify-content   

MDN-Docs on `justify-content`: 
- https://developer.mozilla.org/en-US/docs/Web/CSS/justify-content

---------------------------------------------

### `align-items`: Changing the alignment of flex items on the flex cross axis (per default - i.e. `flex-direction: row` - that's the alignment on the y-axis - so horizontal alignment. Once you change the flex-direction to column instead, the content is aligned along the x-axis - so vertical alignment) 

- `.align-items-start`
- `.align-items-end`
- `.align-items-center`
- `.align-items-baseline`
- `.align-items-stretch`
- + responsive class-variations of the above (i.e. `align-items-md-start`, `align-items-xl-end` etc.)     

```
<div class="d-flex align-items-center bd-highlight mb-3" style="height: 100px">
 <div class="p-2 bd-highlight">Flex item</div>
 <div class="p-2 bd-highlight">Flex item</div>
 <div class="p-2 bd-highlight">Flex item</div>
</div>
```
BS-Docs on `align-items`: 
- https://getbootstrap.com/docs/5.0/utilities/flex/#align-items
MDN-Docs on `align-items`: 
- https://developer.mozilla.org/en-US/docs/Web/CSS/align-items

---------------------------------------------

### Using Auto-margins with Flexbox   

#### Horizontal alignment with `ml-auto` + `mr-auto`

Use one of the following BS-classes to fill the available space left or right of an flex-item with an auto-margin to the left or right - thus pushing the other flex-items in that direction:

- `.ml-auto` (or `ms-auto` in BS 5 - for "margin start")
- `.mr-auto` (or `me-auto` in BS 5 - for "margin end")
 
```
<div class="d-flex bd-highlight mb-3">
  <div class="mr-auto p-2 bd-highlight">Flex item</div>
  <div class="p-2 bd-highlight">Flex item</div>
  <div class="p-2 bd-highlight">Flex item</div>
</div>

<div class="d-flex bd-highlight mb-3">
  <div class="p-2 bd-highlight">Flex item</div>
  <div class="p-2 bd-highlight">Flex item</div>
  <div class="ml-auto p-2 bd-highlight">Flex item</div>
</div>
```

#### Vertical alignment with `mb-auto` + `mt-auto`

Use one of the following BS-classes (in combination with `flex-column` and `align-items-start|end`) to fill the available space above or below of an flex-item with an auto-margin - thus pushing the other flex-items to the top or bottom:

- `.mt-auto`  
- `.mb-auto`  
 
Example from the BS-docs:

```
<div class="d-flex align-items-start flex-column bd-highlight mb-3" style="height: 200px;">
  <div class="mb-auto p-2 bd-highlight">Flex item</div>
  <div class="p-2 bd-highlight">Flex item</div>
  <div class="p-2 bd-highlight">Flex item</div>
</div>

<div class="d-flex align-items-end flex-column bd-highlight mb-3" style="height: 200px;">
  <div class="p-2 bd-highlight">Flex item</div>
  <div class="p-2 bd-highlight">Flex item</div>
  <div class="mt-auto p-2 bd-highlight">Flex item</div>
</div>
```


BS-Docs on flexbox + auto-margins: 
- https://getbootstrap.com/docs/4.6/utilities/flex/#auto-margins


Docs on all relevant BS-flexbox-alignment-features: 
- https://getbootstrap.com/docs/5.0/utilities/flex/#justify-content   
- https://getbootstrap.com/docs/5.0/utilities/flex/#align-items
- https://getbootstrap.com/docs/4.6/utilities/flex/#auto-margins

---------------------------------------------

#### TODO: 
- float-start + float-end (or in BS5: float-start and float-end)
- + responsive class-variations of the above (i.e. `float-md-start`, `float-xl-end` etc.)    

