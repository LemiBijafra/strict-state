<script lang="ts">
	import {
		SvelteFlow,
		Controls,
		MiniMap,
		Position,
		type Node,
		type Edge,
		type XYPosition,
		Background
	} from '@xyflow/svelte';
	import ResizableNodeSelected from './ResizableNodeSelected.svelte';
	import CustomResizerNode from './CustomResizerNode.svelte';
	import '@xyflow/svelte/dist/style.css';
	import type { IExeStateMachine } from '../state-machine-cat/src/exec/index.mjs';
	import { sm, smSrc, updateAst, statechart } from './lib/sm-state.svelte';

	const nodeTypes = {
		selectorNode: CustomResizerNode,
		customResizerNode: CustomResizerNode
	};
	let initPos;
	let dim;
	function initNodes() {
		statechart.n = new Array<Node>();
		statechart.e = new Array<Edge>();
		initPos = { x: 0, y: 0 };
		dim = 400;
	}

	function addNodes(
		statemachine: IExeStateMachine,
		out: Node[],
		parentPos: XYPosition,
		parentDim: number,
		parentId: string | null
	) {
		// nodes (states)
		for (let prop = 0; prop != statemachine.states.length; ++prop) {
			const childDim = parentDim / 2;
			const childPos = { x: parentPos.x + 10 + prop * 2, y: parentPos.y };
			const id = statemachine.states[prop].name;
			//console.log(statemachine.states[prop].name)
			let node = {
				id: id,
				//type: 'default',
				//type: 'ResizableNodeSelected',
				type: 'customResizerNode',
				//type: 'selectorNode',

				data: { label: id },
				position: childPos,
				width: childDim,
				height: childDim
			};
			if (parentId) {
				node.parentId = parentId;
			}
			out.push(node);

			if (statemachine.states[prop].hasOwnProperty('statemachine')) {
				addNodes(
					statemachine.states[prop].statemachine as IExeStateMachine,
					out,
					childPos,
					childDim,
					id
				);
			}
		}
		// edges (transitions)
		if (statemachine.hasOwnProperty('transitions') && statemachine.transitions) {
			for (let prop = 0; prop != statemachine.transitions.length; ++prop) {
				let edge = {
					id: statemachine.transitions[prop].id,
					source: statemachine.transitions[prop].from,
					target: statemachine.transitions[prop].to
				};
				if (statemachine.transitions[prop].label) {
					edge.label = statemachine.transitions[prop].label;
				}
				statechart.e.push(edge);
			}
		}
	}
	export function update() {
		console.log(smSrc.triggerUpdate);
		initNodes();
		updateAst();
		addNodes(sm, statechart.n, initPos, dim, null);
		//smSrc.triggerUpdate = !smSrc.triggerUpdate; // why does this work only every second click?
		smSrc.text = smSrc.text;
	}
</script>

<!-- {#key smSrc.triggerUpdate} -->
{#key smSrc.text}
	<SvelteFlow {nodeTypes} bind:nodes={statechart.n} bind:edges={statechart.e} fitView>

		<div class="updatenode__controls">
			<button
				onclick={() => {
					update();
				}}>Update</button
			>
		</div>
		<Background />
		<Controls />
		<MiniMap />
	</SvelteFlow>
{/key}

<!-- <label>label:</label>
				<input value={''} />

				<label class="updatenode__bglabel">background:</label>
				<input value={''} /> -->
<!-- <div class="updatenode__checkboxwrapper">
					<label>hidden:</label>
					<input
						type="checkbox"
						checked={nodeHidden}
						on:input={(evt) => (nodeHidden = evt.target?.checked)}
					/>
				</div> -->

<style>
	:global(.updatenode__controls) {
		position: absolute;
		right: 10px;
		top: 10px;
		z-index: 4;
		font-size: 12px;
	}

	:global(.updatenode__controls label) {
		display: block;
	}

	:global(.updatenode__bglabel) {
		margin-top: 10px;
	}

	:global(.updatenode__checkboxwrapper) {
		margin-top: 10px;
		display: flex;
		align-items: center;
	}
</style>
