# Basics

## Text and javascript interpolation

Any valid js expression can be transformed to plain text in a template by wrapping it in moustaches. The content of the moustaches strictly has to be an expression, a if else structure is not valid as it is not an expression. [[Vue Summary#Text interpolation]]

```html
<div>{{myObject}}</div>
```

```js
myObject = {
  name: "guillermo",
  surname: "Vicente",
};
```

## Attribute binding

Attributes can be bound with the `v-bind:Attribute name` directive or its shorthand `:attributeName`

```html
<input :type="mYvar" />
```

```js
mYvar = "input";
```

This also applies to custom component props [[Vue Summary#Attribute bindings]]

## Listening to events

To capture events we use the v-on:eventName directive or its syntax sugar @eventName

```html
<button @click="count++">Add 1</button>
<!--Is the same as-->
<button v-on:click="count++"></button>
```

Event handler expect a function callback so we can either define one on the spot, pass a function declaration, or pass a js valid expression.

```html
<button @click="()=>{count++}">Add 1</button>
<button v-on:click="count++"></button>
<button @click="myCallback"></button>
```

```js
function myCallback() {
  count.value++;
}
```

The v-on directive returns an event object we can handle it in different ways. we can either do it explicitly

```html
<button @click="(e)=>handler(e)"></button>
```

Or implicitly

```html
<button @click="hanlder"></button>
```

Where our handler function always is

```js
function greet(event) {
  // `event` is the native DOM event
  if (event) {
    alert(event.target.tagName);
  }
}
```

## Reactivity

Reactivity is achieved with the `reactive()` and `ref()` api calls, ref being the recommended way.
Reactive states are tracked by vue and changes to them trigger re renders so that their updated values are used.

- `Ref`: Takes an argument and returns it rwapped inside a ref object. To access and change the value of a ref object it is necessary to access its .value property
  - `shallowRef`: Refs are deep by default. If we only want to track the surface we use shallow refs
- `reactive`: Can only hold objects, not primitive values. Unlike ref, it is not needed to use a .value property to access the reactive state. [[Vue Summary#Reactivity]]

```js
let count = ref(0);
let myObject = ref({
  nested: {
    count: 0,
  },
  arr: ["foo", "bar"],
});

//BAD!!! We are destroying the ref object
count = 2;
//GOOD !!! We are updating the ref objec's value
count.value = 2;
```

```js
//THIS IS INCORRECT XXXXX
let count = reactive(1);
//This is correct
let state = reactive({ count: 0 });
//THIS IS CORRECT
state.count++;
```

## Computed properties: Memoization

Computed properties are special reactive states. They hold a value that is obtained by evaluating an expression (via callback) but this callback is not evaluated on every render, it is only reevaluated when its dependencies change => memoization

```js
let list = ref([
  { name: "item 1", done: false },
  { name: "item 2", done: false },
  { name: "item 3", done: true },
]);

let computedList = computed(() => {
  return list.value.filter((item) => item.done);
});
```

In this example the computedList is only recalculated when list changes, improving performance by preventing unneeded evaluations

## Watchers (effects)

Watchers track reactive states and perform side effects when they change. Watch is a function that receives 2 parameters, a state to track and a callback to execute

```js
let myVar = ref(0);
watch(myVar, () => {
  console.log("changes have happened in the myVar ref");
});
```

In this example every time myVar changes the console log will be fired.
Reactive state's properties and properties cannot be watched as is, we need a getter function

```js
const obj = reactive({ count: 0 })

watch(
()=>obj.count;
()=>{
//side effect
})
```

### WatchEffect

A special watcher that doesn't need its dependencies to be explicitly defined, it will track the reactive states in the callback function and track them automatically

```js
const todoId = ref(1);
const data = ref(null);

watch(
  todoId,
  async () => {
    const response = await fetch(
      `https://jsonplaceholder.typicode.com/todos/${todoId.value}`
    );
    data.value = await response.json();
  },
  { immediate: true }
);
```

Is transformed into

```js
watchEffect(async () => {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/todos/${todoId.value}`
  );
  data.value = await response.json();
});
```

## 2 Way binding

With the v-model directive we can simplify the data flow of input components.
This way we simplify this

```html
<input :value="text" @input="event => text = event.target.value" />
```

Into this

```html
<input v-model="text" />
```

For each type of form input, v-model will automatically handle the event and property that is desired, for example, with checkboxes v-model uses the checked property and the changed event
``

```html
<input type="checkbox" id="checkbox" v-model="checked" />
<label for="checkbox">{{ checked }}</label>
```

## Template refs

We can get references to html attributes or Vue components with the ref attribute

```html
<input ref="input" />
```

```js
const input = useTemplateRef("my-input");
```

This way we have access to the native methods of an html element (canvas for example) or the methods of a vue class

## Conditional rendering

With the v-if directive we can conditionally render any element. To do so we specify a condition that `v-if` will evaluate to decide if the component should be rendered or not

```html
<template>
  <div v-if="number == 3">El numero es 3</div>
  <div v-else-if="number == 3">EL numero es 4</div>
  <div v-else="">El numero no es 3 ni 4</div>
</template>
```

If we want to group more than one element, templates should be used instead of divs, as they will not appear in the final result (simmiliar to react fragments)

### v-show

It works just like v-if with one main difference. v-if doesn't render the component while v-show only toggles the visitbility css parameter.

## List rendering

`v-for` is the directive for list rendering. That includes arrays, objects etc.

```js
const items = ref([{ message: "Foo" }, { message: "Bar" }]);
```

```html
<li v-for="item in items">{{ item.message }}</li>
```

We can also use an alias to get the index of the current item

```html
<li v-for="(item, index) in items">
  {{ parentMessage }} - {{ index }} - {{ item.message }}
</li>
```

With objects

```html
<ul>
  <li v-for="value in myObject">{{ value }}</li>
</ul>
```

It is really important to use the key attribute because it is used for tracking renders

# Components

[[Vue Summary#Components]]

## Component structure

```html
<template>
  <!--Here goes html code-->
</template>

<script setup lang="ts">
  //Here goes js / ts code. With the lang="ts" we specify typescript
</script>

<style lang="scss" scoped>
  /*Here goes css code*/
</style>
```

## Props declaration

Props are declared with the defineProps compile macro. It has various ways to declare props. [[Vue Summary#Props]]

Plain way

```js
const props = defineProps(["title", "name"]);
```

In depth

```js
const props = defineProps({
  title: String,
  name: {
    type: string,
    required: true,
  },
});
```

With typescript type annotations

```js
defineProps<{
  title?: string
  likes?: number
}>()
```

### Accessing props

Props can then be accessed from the script setup by using the returned props object

```js
const props = defineProps(["title", "name"]);
console.log(props.title);
```

### Destructuring props

From version 3.5 onwards it is possible to destructure props as well as pass them to functions without losing reactivity.

```js
const { foo } = defineProps(["foo"]);
```

To watch the prop we need a getter function

```js
watch(
  () => foo,
  () => {
    /*Effect*/
  }
);
```

## Event Declaration

Events are declared with the defineEmits compiler macro, that returns an emit object. Then they are triggered by using that function with the event name as parameter

```js
const emit = defineEmits(["title-updated"]);

function handleClick() {
  emit("title-updated");
}
```

To pass arguments to an event we pass them to the emit function after specifying the name

```js
let variable = "test";
emit("eventName", variable);
```

And catch them in the same way we handle native events.

```html
<myComponent @eventName="(param)=>handler(param)" />
<!--Or-->
<myComponent @eventName="handler" />
```

### Event typing

If using typescript, events can be described to specify its return values.

```ts
const emit = defineEmits<{
  (e: "change", id: number): void;
  (e: "update", value: string): void;
}>();
```

## One way data flow

The way information flows in vue (and in general) is:

- from parent to child: props
- from child to parent: events

```js
<button @click="handleClick">Update</button>
```

```js
const emit = defineEmits(["post-updated"]);

function handleClick(event) {
  emit("post-updated"); //we fire the event
}
```

```html
<!--Parent component-->
<ChildComponent @post-updated="handleUpdate" />
```

## v-model in custom components

The defineModel compiler macro creates a v-model prop that implements 2 way binding

```js
const model = defineModel();
```

```html
<Child v-model="countModel" /> <span>current count is: {{countModel}} </span>
```

Now we can safely mutate the model from inside the component as well as from the parent.
This is analogous to

```html
<input
  :value="props.modelValue"
  @input="emit('update:modelValue', $event.target.value)"
/>
```

```js
const props = defineProps(["modelValue"]);
const emit = defineEmits(["update:modelValue"]);
```

### Options

An options object can be passed. If we specify a name for the v-model, the options object goes after

```js
const title = defineModel("title", { required: true });
```

```js
const title = defineModel({ required: true });
```

### Multiple v-model bindings

By giving v-models name we can use more than one by component

```js
const firstName = defineModel("firstName");
const lastName = defineModel("lastName");
```

```html
<!--Parent component-->
<MyComponent v-model:first-name="first" v-model:last-name="last"></MyComponent>
```

## Style encapsulation

- `scoped`: The classes will be applied only to the component, they are transformed when transpiled to make sure no styles permeate
  - If we wanted a scoped style to permeate we can tell it to what elements we want it to permeate using the `:deep` pseudo selector
- `module:` The style tag is compiled as css modules and individual classes can be used by binding the `$style.nameOfClass` object

```html
<template>
  <p :class="$style.red">This should be red</p>
</template>

<style module>
  .red {
    color: red;
  }
</style>
```
