<svelte:options tag="fds-gradient-slider"></svelte:options>
<script>
	import { clamp } from 'yootils';
    import {get_current_component} from 'svelte/internal'
    let host = get_current_component()



    let start = 0
	let end = 1.0
    export let value="0;1.0"
    export let from_color="black"
    export let to_color="red"
    export let min=0
    export let max=1.0
    export let show_min_max="none"
    export let num_handles="two"
	export let value_type="float"
	let leftHandle
	let body
	let slider
    let gCSS=""

    let left=""
    let right=""
    calcGradient()
    $: {
        if (from_color || to_color) calcGradient()
        if (value) {
            let arr=value.split(";")
			if (value_type==="float") {
				left=parseFloat(arr[0])		//.toFixed(2)
				right=parseFloat(arr[1])	//.toFixed(2)	
				min=parseFloat(min)		//.toFixed(2)	
				max=parseFloat(max)		//.toFixed(2)	

			} else {
				left=parseInt(arr[0])
				right=parseInt(arr[1])		
				min=parseInt(min)	
				max=parseInt(max)							
			}

            let range=max-min
            start=(left-min)/range
            end=(right-min)/range
        }
    }
    function updateValue() {
        let range=max-min
		if (value_type==="float") {
			left=parseFloat(start*range+min)	//.toFixed(2)
			right=parseFloat(end*range+min)	//.toFixed(2)
		} else {
			left=parseInt(start*range+min)
			right=parseInt(end*range+min)			
		}


        value=left+";"+right
        if (num_handles!=="two") value=""+left+""
    }
    function calcGradient() {
        gCSS="background: linear-gradient(to right, "+from_color+", "+to_color+");"
    }
	function draggable(node) {
		let x;
		let y;
		function handleMousedown(event) {
			if (event.type === 'touchstart') {
				event = event.touches[0];
			}
			x = event.clientX;
			y = event.clientY;
			node.dispatchEvent(new CustomEvent('dragstart', {
				detail: { x, y }
			}));
			window.addEventListener('mousemove', handleMousemove);
			window.addEventListener('mouseup', handleMouseup);
			window.addEventListener('touchmove', handleMousemove);
			window.addEventListener('touchend', handleMouseup);
		}
		function handleMousemove(event) {
			if (event.type === 'touchmove') {
				event = event.changedTouches[0];
			}
			const dx = event.clientX - x;
			const dy = event.clientY - y;
			x = event.clientX;
			y = event.clientY;
			node.dispatchEvent(new CustomEvent('dragmove', {
				detail: { x, y, dx, dy }
			}));
		}
		function handleMouseup(event) {
			x = event.clientX;
			y = event.clientY;
			node.dispatchEvent(new CustomEvent('dragend', {
				detail: { x, y }
			}));
			host.dispatchEvent(new CustomEvent('change', {detail: {value:value}}))

			window.removeEventListener('mousemove', handleMousemove);
			window.removeEventListener('mouseup', handleMouseup);
			window.removeEventListener('touchmove', handleMousemove);
			window.removeEventListener('touchend', handleMouseup);
		}
		node.addEventListener('mousedown', handleMousedown);
		node.addEventListener('touchstart', handleMousedown);
		return {
			destroy() {
				node.removeEventListener('mousedown', handleMousedown);
				node.removeEventListener('touchstart', handleMousedown);
			}
		};
	}
	function setHandlePosition (which) {
		return function (evt) {
			const { left, right } = slider.getBoundingClientRect();
			const parentWidth = right - left;
			const p = Math.min(Math.max((evt.detail.x - left) / parentWidth, 0), 1);
			if (which === 'start') {
				start = p;
				end = Math.max(end, p);
			} else {
				start = Math.min(p, start);
				end = p;
			}
            updateValue()
		}
	}
	function setHandlesFromBody (evt) {
		const { width } = body.getBoundingClientRect();
		const { left, right } = slider.getBoundingClientRect();
		const parentWidth = right - left;
		const leftHandleLeft = leftHandle.getBoundingClientRect().left;
		const pxStart = clamp((leftHandleLeft + evt.detail.dx) - left, 0, parentWidth - width);
		const pxEnd = clamp(pxStart + width, width, parentWidth);
		const pStart = pxStart / parentWidth;
		const pEnd = pxEnd / parentWidth;
		start = pStart;
		end = pEnd;
	}
</script>

<div class="double-range-container">
    {#if show_min_max==="show"}
        <div class="min_value">{min}</div>
        <div class="max_value">{max}</div>
    {/if}
	<div class="slider" bind:this={slider} style={gCSS}>
		<div
			class="body"
			bind:this={body}
			use:draggable
			on:dragmove|preventDefault|stopPropagation="{setHandlesFromBody}"
			style="
				left: {100 * start}%;
				right: {100 * (1 - end)}%;
			"
			></div>
		<div
			class="handle"
			bind:this={leftHandle}
			data-which="start"
			use:draggable
			on:dragmove|preventDefault|stopPropagation="{setHandlePosition('start')}"
			style="
				left: {100 * start}%
			"
		><div class="handleText">
			{#if value_type==="integer"}{left}{:else}{parseFloat(left).toFixed(2)}{/if}
		</div></div>
        {#if num_handles==="two"}
            <div
                class="handle"
                data-which="end"
                use:draggable
                on:dragmove|preventDefault|stopPropagation="{setHandlePosition('end')}"
                style="
                    left: {100 * end}%
                "
            ><div class="handleText">
				{#if value_type==="integer"}{right}{:else}{parseFloat(right).toFixed(2)}{/if}
			</div></div>
        {/if}
	</div>
</div>

<style>
	.double-range-container {
		width: 100%;
		height: 20px;
		user-select: none;
		box-sizing: border-box;
		white-space: nowrap;
        position: relative;
	}
    .min_value {
        position: absolute;
        top: -10px;
        left: 0;
        color: white;
       
    }
    .handleText {
        position: absolute;
        top: 13px; 
        left: 0px;
        color: white;
        font-size: 14px;
        display:none;
    }
    .handle:active .handleText {
        display:block;
    }
    .max_value {
        position: absolute;
        right: 0;
        top: -10px;
        color: white;
    }
	.slider {
		position: relative;
		width: 100%;
		height: 12px;
		top: 50%;
		transform: translate(0, -50%);
		
		border-radius: 1px;
	}
    
    .handle {
    position: absolute;
    top: 12px;
    margin-left: -5px;
    width: 0;
    height: 0;
    transform: translate(-50%, -50%);
    cursor: pointer;
}

.handle:after {
    content: '';
    position: absolute;
    left: 0;
    top: 0;
    width: 0;
    height: 0;
    border-left: 5px solid transparent; /* Left side of the arrow */
    border-right: 5px solid transparent; /* Right side of the arrow */
    border-bottom: 15px solid #ffffff; /* Arrow pointing upwards */
}

	/* .handle[data-which="end"]:after{
		transform: translate(-100%, -50%);
	} */
	.handle:active:after {
        border-bottom: 15px solid #ddb74f;
	}

</style>