# vue-clipper

## Overview

`vue-clipper` is a vue component for taking screenshot that can be wrapped around any element. Supports scrolling while dragging to take a screenshot.

**Reference:** https://www.digitalocean.com/community/tutorials/vuejs-screenshot-ui

## Run Demo
```sh
npm install
npm run serve
```

## Usage

Copy the `clipper.vue` from `src/lib` and paste somewhere inside your project.

```html
<template>
	<div>
    <clipper
    	:width="500px"
      :height="300px"
      @image-cropped="doSomething"       
    >
			<!-- your elements -->
    </clipper>
  </div>
</template>
<script>
import clipper from 'path/to/the/component/clipper';
  
export default {
  components: {
    clipper,
  },
  methods: {
    doSomething(img) {
      let {
        url, // image url for clip
        blob // image blob
      } = img;
      // ...
		},
	},
};
</script>
```

## Screenshots

### Capturing

![capturing](./public/capturing.png)

### Result

![result](./public/result.png)