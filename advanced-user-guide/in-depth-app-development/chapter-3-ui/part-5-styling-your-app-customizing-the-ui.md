---
description: >-
  In this part, you will learn how you can customize HTML in your application.
  We will use our Application Designer to preview the HTML files.
---

# Part 5 — Styling your app \[Customizing the UI]

### Table of contents

1. [Element widgets, again](part-5-styling-your-app-customizing-the-ui.md#step-1-element-widgets-again)
2. [HTML Styles](part-5-styling-your-app-customizing-the-ui.md#step-2-html-styles)
3. [Elements properties (Disabling, Hiding, Loading, Iterating)](part-5-styling-your-app-customizing-the-ui.md#step-3-elements-properties-disabling-hiding-loading-iterating)

### Step 1 — Element widgets, again

Remember the [Part 2 from Chapter 2 — Element widgets](../chapter-2-modal-window/#table-of-contents)?\
We can use elements from [Element](https://element.eleme.io/1.4/#/en-US/component/button) when creating applications with UI.\
It is enough to add the desired element to the HTML file.

<img src="https://github.githubassets.com/images/icons/emoji/unicode/26a0.png" alt="warning" data-size="line"> Don't forget to initialize `state` so the buttons work properly

**Example:**

```html
<div>
	<div>
		<el-button type="success">Success</el-button>
        	<el-button type="warning">Warning</el-button>
        	<el-button type="danger">Danger</el-button>
        	<el-button type="info">Info</el-button>
	</div>
	<hr/>
	<div>
		state.checkedList value: {{state.checkedList}}
	</div>
	<hr/>
	<div>

		<el-checkbox-group v-model="state.checkedList">
		    <el-checkbox label="Option A"></el-checkbox>
		    <el-checkbox label="Option B"></el-checkbox>
		    <el-checkbox label="Option C"></el-checkbox>
		</el-checkbox-group>
	</div>
</div>
```

**Result:**

![](https://github.com/supervisely-ecosystem/how-to-create-app/raw/master/chapter-03-ui/part-05-styling-your-app/media/element-state-init.png)

***

![](https://github.com/supervisely-ecosystem/how-to-create-app/raw/master/chapter-03-ui/part-05-styling-your-app/media/element-example.png)

### Step 2 — HTML Styles

Also you can create your own styles and apply them to elements.\
Inline styles are also available.

**Example:**

```html
<div id="styling-your-app">
    <sly-style>
        #styling-your-app .our-custom-class {
        font-size: 50px;
        }
	</sly-style>

	<div class="our-custom-class">
		Text with custom class
	</div>
	<hr/>
	<div style="font-size: 24px">
		Text with custom style
	</div>
	<hr/>
	<div>
		Text without custom class
	</div>
</div>
```

**Result:**

![](https://github.com/supervisely-ecosystem/how-to-create-app/raw/master/chapter-03-ui/part-05-styling-your-app/media/custom-style-example.png)

### Step 3 — Elements properties (Disabling, Hiding, Loading, Iterating)

Some elements of the HTML file **have properties**.\
They can be changed.

<img src="https://github.githubassets.com/images/icons/emoji/unicode/26a0.png" alt="warning" data-size="line"> You can use `state` to pass values to these properties!

```html
<div>
	<hr/>  <!-- disabling property -->
	<el-button type="success" :disabled="true">i'm disabled</el-button>

	<hr/> <!-- hiding(v-if) property -->
	<el-button type="success" v-if="false">i'm hidden</el-button>
	<el-button type="success" v-if="true">i'm not hidden</el-button>

	<hr/> <!-- loading property -->
	<el-button type="success" :loading="true">i'm loading</el-button>

	<hr/> <!-- iterating property -->
	<!--
	fill the data field:
	{"simpleData":
		{
			"now": "we",
			"can": "iterate",
			"in": "HTML!"
		}
	}
	-->

	<div v-for="v,k in data.simpleData" style="word-break: break-all;">
    		<b>{{k}}</b>: <span>{{v}}</span>
	</div>
</div>
```

**Result:**

![](https://github.com/supervisely-ecosystem/how-to-create-app/raw/master/chapter-03-ui/part-05-styling-your-app/media/elements-props-data.png)

***

![](https://github.com/supervisely-ecosystem/how-to-create-app/raw/master/chapter-03-ui/part-05-styling-your-app/media/elements-props-preview.png)
