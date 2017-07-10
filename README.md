# Datepicker

# simple-vue-datepicker
Implement a very simple calendar-based datepicker

Derived and stripped down from https://github.com/charliekassel/vuejs-datepicker


A datepicker Vue component. Compatible with Vue 2.x

N.B. - this is in a very raw intermediate state and is not meant for general public consumption at this point. I'm working on another project and will be cleaning this up over time but for now:

1. it's not tested except for my specific use cases
2. the documentation is inaccurate - it has not been scrubbed
3. it's only partially stripped down - there are lots of relics

## Install

``` bash
$ npm install simple-vue-datepicker --save
```
``` javascript
import Datepicker from 'simple-vue-datepicker';

Vue.component('my-component', {
    components: {
        Datepicker
    }
});
```


## Usage

``` html
<datepicker></datepicker>
```

*value* prop sets the selectedDate. should be a Date object. defaults to null - no date selected.

``` html
<script>
var state = {
    date: new Date(2016, 9,  16)
}
</script>
<datepicker :value="state.date"></datepicker>
```

Use `v-model` for two-way binding
``` html
<datepicker v-model="state.date"></datepicker>
```

Emits events

Some of these might still work
``` html
<datepicker v-on:selected="doSomethingInParentComponentFunction" v-on:opened="datepickerOpenedFunction" v-on:closed="datepickerClosedFunction">
```
Inline always open version
``` html
<datepicker :inline="true"></datepicker>
```
## Available props

| Prop                  | Type         | Default     | Description                              |
|-----------------------|--------------|-------------|------------------------------------------|
| value                 | Date/String  |             | Date value of the datepicker             |
| language              | String       | en          | Translation for days and months          |
| disabled              | Object       |             | See below for configuration              |
| inline                | Boolean      |             | To show the datepicker always open       |
| initialViewDate       | Date/String  | value       | Year/month to initially show on calendar |

## Events

These events are emitted on actions in the datepicker

| Event         | Output     | Description                   |
|---------------|------------|-------------------------------|
| opened        |            | The picker is opened          |
| closed        |            | The picker is closed          |
| selected      | Date\|null | A date has been selected      |
| cleared       |            | Selected date has been cleared|


#### Disabled Dates
Dates can disabled in a number of ways.

``` html
<script>
var state = {
    disabled: {
        to: new Date(2016, 0, 5), // Disable all dates up to specific date
        from: new Date(2016, 0, 26), // Disable all dates after specific date
        days: [6, 0], // Disable Saturday's and Sunday's
        dates: [ // Disable an array of dates
            new Date(2016, 9, 16),
            new Date(2016, 9, 17),
            new Date(2016, 9, 18)
        ]
    }
}
</script>
<datepicker :disabled="state.disabled"></datepicker>
```

#### Highlight Dates
Dates can be highlighted (e.g. for marking an appointment) in a number of ways. Important: You can only highlight dates that aren't disabled.
Note: Both `to` and `from` properties are required to define a range of dates to highlight

``` html
<script>
var state = {
    highlighted: {
        to: new Date(2016, 0, 5), // Highlight all dates up to specific date
        from: new Date(2016, 0, 26), // Highlight all dates after specific date
        days: [6, 0], // Highlight Saturday's and Sunday's
        dates: [ // Highlight an array of dates
            new Date(2016, 9, 16),
            new Date(2016, 9, 17),
            new Date(2016, 9, 18)
        ]
    }
}
</script>
<datepicker :highlighted="state.highlighted"></datepicker>
```


#### Translations

Contributing guide - please use appropriate code from this [list](http://www.iana.org/assignments/language-subtag-registry/language-subtag-registry) as the translation property.

``` html
<datepicker language="es"></datepicker>
```
Available languages

| Abbr        | Language         |          |
| ----------- |------------------|----------|
| ar          | Arabic           |          |
| bg          | Bulgarian        |          |
| bs          | Bosnian          |          |
| cs          | Czech            |          |
| da          | Danish           |          |
| de          | German           |          |
| ee          | Estonian         |          |
| en          | English          | *Default*|
| es          | Spanish          |          |
| fi          | Finnish          |          |
| fr          | French           |          |
| he          | Hebrew           |          |
| hu          | Hungarian        |          |
| hr          | Croatian         |          |
| id          | Indonesian       |          |
| is          | Icelandic        |          |
| it          | Italian          |          |
| ja          | Japanese         |          |
| ko          | Korean           |          |
| lt          | Lithuanian       |          |
| nb-no       | Norwegian Bokmål |          |
| nl          | Dutch            |          |
| pl          | Polish           |          |
| pt-br       | Portuguese-Brazil|          |
| ro          | Romanian         |          |
| ru          | Russian          |          |
| sk          | Slovak           |          |
| sl-si       | Slovenian        |          |
| sv          | Swedish          |          |
| th          | Thai             |          |
| tr          | Turkish          |          |
| uk          | Ukrainian        |          |
| vi          | Vietnamese       |          |
| zh          | Chinese          |          |



