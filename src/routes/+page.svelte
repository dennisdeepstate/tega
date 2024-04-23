<script lang="ts">
	import { browser } from '$app/environment'

	let words = [
		{ id: 1, word: 'beautiful', definition: 'a girl' },
		{ id: 1, word: 'handsome', definition: 'a boy' }
	]
	let inputs: HTMLInputElement[] = browser
		? Array.from(document.querySelectorAll('input[data-index]'))
		: []
	let letter_bindings: string[] = []
	let user_values: (string | undefined)[] = []

	const bad_letters = [' ', '/', "'"]

	function create_level(
		words: { id: number; word: string; definition: string; synonym?: string | null }[]
	) {
		let input_index = -1
		return words.map((word) => {
			const letters = word.word.split('').filter((letter) => !bad_letters.includes(letter))
			const bound_letters = letters.map((letter) => {
				input_index += 1
				let binding_index: number
				const matching_binding_index = letter_bindings.findIndex((binding) => binding === letter)
				if (matching_binding_index < 0) {
					letter_bindings.push(letter)
					user_values.push(undefined)
					binding_index = letter_bindings.length - 1
				} else {
					binding_index = matching_binding_index
				}

				return {
					binding_index,
					letter,
					input_index
				}
			})

			return {
				bound_letters,
				word
			}
		})
	}

	function focus_previous({ index }: { index: number }) {
		let next_element_index = index - 1
		if (next_element_index < 0) return
		inputs[next_element_index].focus()
	}

	function focus_next({ index }: { index: number }) {
		if (!inputs[index] || !inputs[index].value.length) return
		let next_element_index = index + 1
		if (next_element_index >= inputs.length) {
			const empty_value_index = inputs.findIndex((input) => !Boolean(input.value.length))
			if (empty_value_index < 0) return
			next_element_index = empty_value_index
		}
		if (inputs[next_element_index].value.length) {
			focus_next({ index: next_element_index })
			return
		}
		inputs[next_element_index].focus()
	}

	function handle_key({
		event,
		index
	}: {
		event: KeyboardEvent & { currentTarget: EventTarget & HTMLInputElement }
		index: number
	}) {
		if (event.key !== 'Backspace' && event.type === 'keyup') focus_next({ index })
		if (event.key === 'Backspace' && !inputs[index].value.length && event.type === 'keydown') {
			focus_previous({ index })
			event.preventDefault()
		}
	}

	const level = create_level(words)
</script>

<div class="container h-full mx-auto flex justify-center items-center">
	<div class="space-y-5">
		<h1 class="h1">Tega</h1>
		<form>
			{#each level as word}
				<div class="grid grid-cols-1">
					<div class="relative">
						{#each word.bound_letters as bound_letter}
							<label class="inline-block">
								{bound_letter.binding_index + 1}
								<input
									class="input w-16 p-2 rounded-none text-center"
									on:keydown={(e) => handle_key({ event: e, index: bound_letter.input_index })}
									on:keyup={(e) =>
										handle_key({
											event: e,
											index: bound_letter.input_index
										})}
									bind:value={user_values[bound_letter.binding_index]}
									maxlength="1"
									data-index={bound_letter.input_index}
									bind:this={inputs[bound_letter.input_index]}
								/>
							</label>
						{/each}
					</div>
					<span>{word.word.definition}</span>
				</div>
			{/each}
		</form>
	</div>
</div>
