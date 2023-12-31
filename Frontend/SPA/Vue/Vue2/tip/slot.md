# slot

```
//sectionWrapComp.vue
<template>
	<div>
		<h2>{{title}}</h2>
		<slot name="content"></slot>
	</div>
</template>
props: [
	title,
	index,
	hasRemove
]
//End of sectionWrapComp.vue


//itemWrapComp.vue
<template>
	<div>
		<lable>{{name}}</label>
		<span v-if="description">{{description}}</span>
		<slot name="content"></slot>
	</div>
</template>
props: [
	name,
	description,
]
//End of itemWrapComp.vue


//resume.vue
<template>
	<section-wrap v-for="(item, idx) in data" title="기본정보" :index="idx">
		<item-wrap name="잡 타이틀" description="전반적인 직무 경험이나....">
			<template v-slot:content>
				<sf-input />
			</template>
		</item-wrap>
	</section-wrap>
</template>
componets: {
	section-wrap: sectionWrapComp,
	item-wrap: itemWrapComp,
	sf-input: sfInput,
}


<div>
	<h2>기본정보</h2>
	<div>
		<lable>잡 타이틀"</label>
		<span >전반적인 직무 경험이나....</span>
		<input type="text" />
	</div>
</div>
```

