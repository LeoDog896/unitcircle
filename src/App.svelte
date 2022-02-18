<script lang="ts">
	import "mathlive"
	import { renderMathInDocument } from 'mathlive';
	import { ComputeEngine, Expression, serialize } from '@cortex-js/compute-engine';
	import Tailwindcss from "./Tailwindcss.svelte"
	renderMathInDocument()

	function getRandomWholeNumber(min: number, max: number) {
		min = Math.ceil(min);
		max = Math.floor(max);
		return Math.floor(Math.random() * (max - min + 1)) + min;
	}


	type TrigFunction = "sin" | "cos" | "tan" | "csc" | "sec" | "cot"

	const trigFunctions: {
		[key in TrigFunction]: (number: number) => number
	} = {
		sin: Math.sin,
		cos: Math.cos,
		tan: Math.tan,
		csc: number => 1 / Math.sin(number),
		sec: number => 1 / Math.cos(number),
		cot: number => 1 / Math.tan(number)
	}

	function generateProblem(): Problem {
		const key = Object.keys(trigFunctions)[Math.floor(Math.random() * Object.keys(trigFunctions).length)] as TrigFunction
		const ce = new ComputeEngine();

		const simplifiedProblem = ce.simplify(
			[
				key,
				["Divide", ["Multiply", getRandomWholeNumber(1, 12), "Pi"], "6"]
			],
			{
				simplifications: ["simplify-arithmetic"]
			}
		)

		return {
			baseName: key,
			base: trigFunctions[key],
			equation: simplifiedProblem
		}
	}
	
	interface Problem {
		baseName: TrigFunction,
		base: (number: number) => number,
		equation: Expression
	}

	let showField;
	let currentProblem = generateProblem()

</script>
<math-field class="w-1/4" bind:this={showField} read-only>{serialize(currentProblem.equation)}</math-field>
<math-field class="w-1/4" virtual-keyboard-mode=manual>
	\pi
</math-field>
<button on:click={() => {
	currentProblem = generateProblem();
	showField.setValue(
		serialize(currentProblem.equation),
		{ suppressChangeNotification: true }
	)
}}>Check</button>
<Tailwindcss />