# TwoDPlot

Demo:
https://www.youtube.com/watch?v=-3PLtTk1AdE

Usage: 

```
import TWODPlot from './components/TwoDPlot.svelte';
```
```
let data_points=[];
```

```
			<TWODPlot style="display: block;"
				_width="600"
				_height="600"
				bind:data_array={data_points}
				format="line"
			></TWODPlot>
```

